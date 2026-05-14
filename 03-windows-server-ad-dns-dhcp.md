# Windows Server 2022 - Active Directory, DNS, DHCP, and Group Policy

**Cert alignment:** CompTIA A+, CompTIA Network+, CompTIA Security+
**Lab VM:** dc01 (10.10.10.10, VLAN 10)
**Last reviewed:** 2026-07

---

## Installation Notes

Windows Server 2022 Standard Evaluation is available free for 180 days from
Microsoft's Evaluation Centre. The evaluation licence can be reset by running
`slmgr /rearm` for additional evaluation periods during lab use.

After installing Windows Server 2022 in the Proxmox VM:
- Install VirtIO guest drivers for best disk/network performance
  (download from `fedorapeople.org/groups/virt/virtio-win/direct-downloads`)
- Set static IP: `10.10.10.10 / 255.255.255.0 / GW: 10.10.10.1 / DNS: 127.0.0.1`
- Set hostname to `dc01` before promoting to domain controller

---

## Active Directory Domain Services

### Promote to Domain Controller

```powershell
# Install the AD DS role and management tools
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller — creates a new forest
# This also installs DNS (integrated with AD — required)
Install-ADDSForest `
    -DomainName             "contoso.local" `
    -DomainNetBiosName      "CONTOSO" `
    -ForestMode             "WinThreshold" `
    -DomainMode             "WinThreshold" `
    -InstallDns:$true `
    -SafeModeAdministratorPassword (
        ConvertTo-SecureString "SafeMode@Lab2026!" -AsPlainText -Force
    ) `
    -Force

# Server will restart automatically after promotion
```

### Verify AD Installation

```powershell
# Confirm the DC is operational
Get-ADDomainController | Select-Object Name, Domain, Forest, IPv4Address

# Verify SYSVOL and NETLOGON shares are present (required for GPOs and logon scripts)
net share | Select-String -Pattern "SYSVOL|NETLOGON"

# Check all AD-related services are running
$ADServices = @("ADWS", "KDC", "Netlogon", "DNS", "NTFRS", "W32Time")
Get-Service $ADServices | Select-Object Name, Status, StartType
```

### Create the Organisational Unit Structure

```powershell
# Create a hierarchical OU structure matching a real enterprise layout
$Domain = "DC=contoso,DC=local"

New-ADOrganizationalUnit -Name "Staff"          -Path $Domain
New-ADOrganizationalUnit -Name "IT"             -Path "OU=Staff,$Domain"
New-ADOrganizationalUnit -Name "Finance"        -Path "OU=Staff,$Domain"
New-ADOrganizationalUnit -Name "Operations"     -Path "OU=Staff,$Domain"
New-ADOrganizationalUnit -Name "Human Resources"-Path "OU=Staff,$Domain"
New-ADOrganizationalUnit -Name "Contractors"    -Path $Domain
New-ADOrganizationalUnit -Name "Service Accounts"-Path $Domain
New-ADOrganizationalUnit -Name "Workstations"   -Path $Domain
New-ADOrganizationalUnit -Name "Disabled Users" -Path $Domain
New-ADOrganizationalUnit -Name "Azure-Sync"     -Path $Domain

Write-Host "OU structure created."
```

### Create Lab User Accounts

```powershell
# Create several test users across OUs for realistic testing
$SecurePW = ConvertTo-SecureString "Welcome@Lab2026!" -AsPlainText -Force

$Users = @(
    @{ First="Jane"; Last="Mwangi"; OU="OU=Finance,OU=Staff"; Dept="Finance"; Title="Accountant" },
    @{ First="Brian"; Last="Otieno"; OU="OU=IT,OU=Staff"; Dept="IT"; Title="IT Support Analyst" },
    @{ First="Amina"; Last="Hassan"; OU="OU=Human Resources,OU=Staff"; Dept="HR"; Title="HR Coordinator" },
    @{ First="David"; Last="Mutua"; OU="OU=IT,OU=Staff"; Dept="IT"; Title="Systems Administrator" },
    @{ First="testuser1"; Last=""; OU="OU=Azure-Sync"; Dept="IT"; Title="Test Account" }
)

foreach ($User in $Users) {
    if ($User.Last -ne "") {
        $Username    = ($User.First.Substring(0,1) + $User.Last).ToLower()
        $DisplayName = "$($User.First) $($User.Last)"
    } else {
        $Username    = $User.First.ToLower()
        $DisplayName = $User.First
    }

    New-ADUser `
        -Name              $DisplayName `
        -GivenName         $User.First `
        -Surname           $User.Last `
        -SamAccountName    $Username `
        -UserPrincipalName "$Username@contoso.local" `
        -Department        $User.Dept `
        -Title             $User.Title `
        -AccountPassword   $SecurePW `
        -Enabled           $true `
        -ChangePasswordAtLogon $false `
        -Path              "$($User.OU),DC=contoso,DC=local"

    Write-Host "Created: $Username ($DisplayName)"
}
```

---

## DNS Configuration

Active Directory DNS is installed and configured automatically during the domain
promotion. The domain controller serves as the authoritative DNS server for
`contoso.local`.

### Verify DNS is Working

```powershell
# Verify the DC can resolve its own hostname
Resolve-DnsName "dc01.contoso.local"

# Verify forward lookup zone exists
Get-DnsServerZone | Where-Object { $_.ZoneName -eq "contoso.local" }

# Verify reverse lookup zone
Get-DnsServerZone | Where-Object { $_.IsReverseLookupZone -eq $true }

# Test that a domain client can be registered
# (Run from win10-client after domain join)
# ipconfig /registerdns
# Resolve-DnsName "win10-client.contoso.local"
```

### Add a Static DNS Record (Example)

```powershell
# Add a static A record (useful for lab services that don't register via DHCP)
Add-DnsServerResourceRecordA `
    -Name        "splunk" `
    -ZoneName    "contoso.local" `
    -IPv4Address "10.10.99.10"

# Verify
Resolve-DnsName "splunk.contoso.local"
```

---
