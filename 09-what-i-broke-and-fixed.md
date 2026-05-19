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

