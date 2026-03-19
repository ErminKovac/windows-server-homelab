# 05 — PowerShell Basics

## Goal
Learn the PowerShell commands most commonly used in day-to-day Windows Server
and Active Directory administration.

---

## Why PowerShell matters

The GUI is fine for learning, but in a real sysadmin job:
- You might need to create 50 user accounts at once, PowerShell does it in seconds
- You might need to check something on a server remotely without RDP
- You might need to automate a daily task so it runs on a schedule

---

## Commands I ran in this lab

### Verified AD users and group membership

```powershell
# List all AD users
Get-ADUser -Filter * | Select-Object Name, SamAccountName, Enabled

# Check IT_Admins group members
Get-ADGroupMember -Identity "IT_Admins" | Select-Object Name

# Check HR_Staff group members
Get-ADGroupMember -Identity "HR_Staff" | Select-Object Name
```

Results confirmed:
- Ermin Kovac (e.kovac) — Enabled — IT_Admins
- Tarik Mehic (t.mehic) — Enabled — IT_Admins
- Almin Skeledzija (a.skeledzija) — Enabled — HR_Staff

### Verified Group Policy

```powershell
# Force immediate Group Policy refresh
gpupdate /force

# View applied GPOs (text report)
gpresult /r

# View applied GPOs (HTML report — open in browser)
gpresult /h C:\GPOReport.html
```

### Verified DNS

```powershell
# Test DNS resolution by name
nslookup WS-LAB.lab.local

# Test DNS resolution using PowerShell
Resolve-DnsName WS-LAB.lab.local
```

Both returned 192.168.1.10 — DNS working correctly.

### Verified DHCP

```powershell
# View all DHCP scopes on the server
Get-DhcpServerv4Scope
```

Returned Lab Network scope — Active — 192.168.1.100 to 192.168.1.200.

### Checked installed roles

```powershell
# List all installed Windows features/roles
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} | Select-Object Name, DisplayName
```

Confirmed installed: AD-Domain-Services, DHCP, DNS, GPMC, RSAT tools.

### Checked disk space

```powershell
# View filesystem drives and free space
Get-PSDrive -PSProvider FileSystem
```

C: drive — 11.34 GB used, 38.05 GB free.

### Checked running services

```powershell
# List all currently running services
Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object Name, Status
```

Confirmed key services running: ADWS, DHCPServer, DNS, Netlogon, NTDS.

---

## Essential cmdlets for AD administration

### Users

```powershell
# Get a specific user with all properties
Get-ADUser -Identity "e.kovac" -Properties *

# Create a new AD user
New-ADUser `
  -Name "Test User" `
  -GivenName "Test" `
  -Surname "User" `
  -SamAccountName "t.user" `
  -UserPrincipalName "t.user@lab.local" `
  -Path "OU=IT_Department,DC=lab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) `
  -Enabled $true

# Disable a user account
Disable-ADAccount -Identity "e.kovac"

# Enable a user account
Enable-ADAccount -Identity "e.kovac"

# Reset a user's password
Set-ADAccountPassword -Identity "e.kovac" `
  -NewPassword (ConvertTo-SecureString "NewP@ss123" -AsPlainText -Force) `
  -Reset

# Unlock a locked-out account
Unlock-ADAccount -Identity "e.kovac"

# Find all disabled accounts
Get-ADUser -Filter {Enabled -eq $false} | Select-Object Name, SamAccountName
```

### Groups

```powershell
# List all groups
Get-ADGroup -Filter * | Select-Object Name, GroupCategory

# Add a user to a group
Add-ADGroupMember -Identity "IT_Admins" -Members "e.kovac"

# Remove a user from a group
Remove-ADGroupMember -Identity "IT_Admins" -Members "e.kovac" -Confirm:$false
```

### Server & System

```powershell
# Check which Windows Server version is running
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion

# Check running services
Get-Service | Where-Object {$_.Status -eq "Running"}

# Restart a specific service
Restart-Service -Name "DNS"

# Get system uptime
(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime
```

### Networking

```powershell
# View IP configuration
Get-NetIPAddress

# Test connectivity
Test-Connection -ComputerName "8.8.8.8" -Count 4

# Test if a specific port is open
Test-NetConnection -ComputerName "WS-LAB" -Port 389

# Flush DNS cache
Clear-DnsClientCache
```
---

## What I learned

PowerShell uses a consistent Verb-Noun pattern (Get-ADUser, New-ADUser,
Set-ADUser, Remove-ADUser) which makes it very learnable once you understand
the pattern. If you know how to Get something, you usually know how to Set it.

The most valuable habit: when in doubt, use Get-Help:
```powershell
Get-Help New-ADUser -Examples
```
