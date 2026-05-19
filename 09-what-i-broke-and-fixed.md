# What I Broke and How I Fixed It

This document records eight real issues encountered during the build of this lab,
the investigation steps taken, the root cause found, and the fix applied.

Documenting failures is more valuable than only documenting successes. Every issue
here represents a genuine diagnostic exercise - the kind of real-world troubleshooting
that separates a candidate who has worked in hands-on environments from one who has
only followed step-by-step tutorials.

---

## Issue 1: Domain Controller Not Responding to DNS Queries from VLAN 10 Clients

**Symptom:**
After configuring pfSense VLANs and setting win10-client's DNS to 10.10.10.10 (dc01),
the client could ping 10.10.10.10 but `nslookup google.com` returned "Server failed"
and `nslookup dc01.contoso.local` also failed.

**Investigation:**
```cmd
# From win10-client
ping 10.10.10.10          # Succeeded — dc01 is reachable at L3
nslookup dc01.contoso.local 10.10.10.10   # Failed — "Server failed"
Test-NetConnection -ComputerName 10.10.10.10 -Port 53   # TcpTestSucceeded: False
```

TCP port 53 was unreachable on dc01, confirming the DNS service was not listening
or was being blocked.

```powershell
# On dc01
Get-Service DNS     # Status: Stopped
```

The DNS Server service had stopped - it had not been started after the last DC restart.

**Root cause:**
The DNS Server service StartType was set to Manual rather than Automatic. This was
caused by a botched early AD installation attempt where I had manually stopped DNS
to troubleshoot something unrelated and changed its startup type. When the VM
restarted, DNS did not start automatically.

**Fix:**
```powershell
# On dc01
Set-Service DNS -StartupType Automatic
Start-Service DNS
Get-Service DNS     # Confirmed Running

# Verify DNS is now accepting queries
nslookup dc01.contoso.local 10.10.10.10   # Succeeded
```

**Lesson learned:**
Always verify service startup type after any manual service interaction. Use
`Get-Service | Where-Object { $_.StartType -ne 'Automatic' -and $_.Status -eq 'Running' }`
to find services running with non-automatic startup - these will not survive a restart.

---

## Issue 2: pfSense Blocking AD Connect Sync Traffic to Azure

**Symptom:**
After configuring pfSense as the default gateway for VLAN 10 (replacing the home
router), Azure AD Connect stopped syncing. The Synchronisation Service Manager
showed the Azure connector in a "stopped-connectivity" state.


**Investigation:**
```powershell
# On dc01
Test-NetConnection -ComputerName "login.microsoftonline.com" -Port 443
# TcpTestSucceeded: False — Azure endpoints unreachable
Test-NetConnection -ComputerName "8.8.8.8" -Port 443
# TcpTestSucceeded: True — general internet HTTPS works
```

General HTTPS worked but specific Microsoft hostnames were failing. This was a DNS
issue — the domains were resolving but the connections were being blocked.


**Checked pfSense firewall logs:**
```
pfSense Web UI → Status → System Logs → Firewall
Filter by: source = 10.10.10.10 (dc01 IP)
```

Found multiple blocked connections from 10.10.10.10 to `login.microsoftonline.com`
and `account.activedirectory.windowsazure.com`. The pfSense default deny rule was
blocking them — the WORK interface rule did not explicitly allow traffic to these
specific destinations.

**Root cause:**
The VLAN 10 (WORK) firewall rule "Allow WORK net → WAN" was configured to allow
all traffic to WAN, but I had accidentally created a more specific deny rule above
it for a category of traffic that matched Microsoft's OAuth endpoints. The more
specific deny rule was evaluated first.

**Fix:**
```
pfSense Web UI → Firewall → Rules → WORK interface
Identified the overly-broad deny rule
Deleted it (after confirming it was not needed for any other purpose)
```

After rule cleanup, AD Connect sync completed within 5 minutes.

**Lesson learned:**
pfSense evaluates rules top-to-bottom - first match wins. A deny rule placed above
a broad allow rule will block matching traffic even if the allow rule would otherwise
permit it. Always review the complete rule list when debugging blocked traffic,
not just the allow rules.

---

## Issue 3: DHCP Not Serving Leases to VLAN 10 Clients

**Symptom:**
win10-client was getting an APIPA address (169.254.x.x) instead of an address from
the 10.10.10.100–200 range. It could not reach the domain controller.

**Investigation:**
```cmd
# From win10-client
ipconfig /all
# IPv4 Address: 169.254.x.x (APIPA — DHCP client gave up)
# Default Gateway: none
ipconfig /release
ipconfig /renew
# Error: "Unable to contact your DHCP server"
```

```powershell
# From dc01 — check DHCP server status
Get-Service DHCPServer | Select-Object Name, Status
# Status: Running

Get-DhcpServerv4Scope | Select-Object Name, State
# State: Active — scope appears correct
```

DHCP was running and the scope was active. The lease requests were not reaching dc01.

**Checked pfSense DHCP relay configuration:**
The VLAN 10 interface on pfSense was NOT configured to relay DHCP broadcasts to
the Windows DHCP server. pfSense was receiving the DHCP broadcast from win10-client
but had no instructions to forward it to dc01. pfSense's own DHCP server was
disabled for VLAN 10 (correct), but without a DHCP relay, the broadcasts went nowhere.

**Root cause:**
Missing DHCP relay configuration on pfSense WORK interface. The Windows DC serves
DHCP for VLAN 10, but pfSense (as the default gateway for VLAN 10) must relay DHCP
broadcasts from clients to the DC. Without the relay, clients cannot reach the DHCP
server on a different subnet.

**Fix:**
```
pfSense Web UI → Services → DHCP Relay
  Enable       : Checked
  Interface(s) : WORK (VLAN 10)
  Destination  : 10.10.10.10 (dc01 — the Windows DHCP server)
  → Save → Apply
```

After enabling DHCP relay, win10-client received a lease from the correct range
within 30 seconds.

**Lesson learned:**
In a routed network (which is what VLANs with a router/firewall create), DHCP
broadcasts do not cross subnet boundaries by default. Whenever a DHCP server is on
a different subnet from its clients, a DHCP relay agent is required on the gateway
device. This is a fundamental networking concept covered in CompTIA Network+
and encountered constantly in enterprise environments.

---

## Issue 4: Splunk Universal Forwarder Not Sending Events

**Symptom:**
After installing the Splunk Universal Forwarder on dc01 and configuring inputs.conf,
no events appeared in Splunk after 15 minutes.

**Investigation:**
```powershell
# On dc01 — check forwarder status
Get-Service SplunkForwarder | Select-Object Name, Status
# Status: Running

# Check forwarder is pointed at the right server
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server
# Output: Active forwards:
#   None   ← The forwarder had no receiving server configured
```

The forwarder had no receiving server configured. The `outputs.conf` file was
missing — the forwarder did not know where to send data.


**Root cause:**
`inputs.conf` defines what data to collect. `outputs.conf` defines where to send it.
I had created `inputs.conf` in the `system/local` directory but had not created
`outputs.conf`. Without outputs configuration, the forwarder collects data but
has nowhere to send it.

**Fix:**
Created `outputs.conf` at:
`C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 10.10.99.10:9997

[tcpout-server://10.10.99.10:9997]
```

Restarted the forwarder:
```powershell
Restart-Service SplunkForwarder
```

Events appeared in Splunk within 60 seconds.

**Lesson learned:**
In Splunk, inputs and outputs are completely separate configuration files.
`inputs.conf` = data sources. `outputs.conf` = data destinations. Both must
be present and correct for data to flow. The Splunk documentation covers both
but it is easy to miss the outputs configuration when focused on inputs.

---

## Issue 5: Nessus Scan Returning Zero Results on VLAN 10 Targets

**Symptom:**
A Nessus scan configured with targets 10.10.10.10 and 10.10.10.50 completed
in 30 seconds with zero findings - not even informational results. A correctly
running scan against Windows hosts should produce at least 50-100 findings.

**Investigation:**
The scan completed instantly, which was suspicious. A real network scan
takes several minutes minimum. The scan log showed "No hosts found."

```bash
# From ubuntu-admin (the Nessus host — 10.10.10.20)
# Test if targets are reachable
ping -c 4 10.10.10.10    # Succeeded
nmap -p 22,80,443,3389 10.10.10.10   # All ports filtered — stealth scan blocked?
```

Ports appeared filtered even though dc01 was reachable. The issue was that the
Nessus credentialed scan was failing silently - uncredentialed scans show as
"complete" with minimal findings when authentication fails.


**Checked Windows Firewall on dc01:**
Windows Defender Firewall was blocking all inbound connections from the ubuntu-admin
IP (10.10.10.20) except the specific ports allowed by the Windows Firewall rules.
Nessus needs ICMP, SMB (445), WMI, and several other ports to perform credentialed scans.

**Root cause:**
The GPO "Workstation-Security-Baseline" had a setting that enabled Windows Firewall
in blocking mode for all inbound traffic not explicitly allowed. dc01 inherited
this policy because it was in the domain root scope, not scoped to the Workstations OU.

**Fix:**
Two steps:
1. Move the Workstation-Security-Baseline GPO to be linked at the Workstations OU only,
   not the domain root — so it applies only to workstations, not servers.
2. Create a Nessus scanning firewall exception rule on dc01:

```powershell
# On dc01 — allow Nessus scanner (ubuntu-admin) to reach WMI and SMB
New-NetFirewallRule `
    -DisplayName "Allow Nessus Scanner" `
    -Direction   Inbound `
    -RemoteAddress "10.10.10.20" `
    -Protocol    TCP `
    -LocalPort   135,139,445 `
    -Action      Allow

New-NetFirewallRule `
    -DisplayName "Allow Nessus ICMP" `
    -Direction   Inbound `
    -RemoteAddress "10.10.10.20" `
    -Protocol    ICMPv4 `
    -Action      Allow
```

After applying these rules, the next Nessus scan returned 87 findings including
several medium-severity missing patch findings. Scan completed in 8 minutes.

**Lesson learned:**
When a vulnerability scan returns zero or trivially few results, the first question
is not "are the hosts secure?" but "can the scanner actually reach the hosts?"
A scanner that cannot authenticate or reach the required ports will report
nothing rather than reporting an error — making zero findings a false negative,
not a clean bill of health.

---

## Issue 6: win10-client Could Not Receive Group Policy After Domain Join

**Symptom:**
After joining win10-client to contoso.local and running `gpupdate /force`, the
command returned success but `gpresult /r` showed "The user does not have RSoP
data" and no policies were listed as applied.


**Investigation:**
```cmd
# On win10-client
gpresult /r
# Result: ERROR: The user does not have RSoP data.

# Check event log for Group Policy processing errors
eventvwr.msc → Applications and Services Logs → Microsoft → Windows → Group Policy → Operational
# Found: Event 1030 — "Windows could not access the file gpt.ini"
# for domain controller dc01.contoso.local
```

The Group Policy engine could not access the SYSVOL share on the domain controller.

```cmd
# Test SYSVOL accessibility from win10-client
net use \\dc01.contoso.local\SYSVOL
# Error: "System error 5 — Access Denied"
```

SMB access to the domain controller's SYSVOL share was being denied.

```powershell
# On dc01 — check SYSVOL share permissions
net share SYSVOL
# Share exists and appears correct

# Check NTFS permissions on SYSVOL
icacls C:\Windows\SYSVOL
# Found: Domain Users were missing Read permission on a subfolder
```

