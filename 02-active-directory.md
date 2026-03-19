# 02 — Active Directory

## Goal
Install Active Directory Domain Services (AD DS), promote the server to a
Domain Controller, and build a realistic OU and user structure like you'd
find in a small company.

---

## Key concepts before starting

| Term | What it means |
|------|--------------|
| **Active Directory (AD)** | Microsoft's directory service — a central database of users, computers, and policies |
| **Domain Controller (DC)** | The server that runs AD and handles authentication for the whole domain |
| **Domain** | A logical group of computers and users managed together (e.g. `lab.local`) |
| **OU (Organisational Unit)** | A folder inside AD used to organise objects and apply Group Policy |
| **Forest** | The top-level container — a company usually has one forest with one or more domains |

---

## Steps I followed

### 1. Installed the AD DS role
- Opened **Server Manager** → Manage → Add Roles and Features
- Role-based installation → selected `WS-LAB-01`
- Checked **Active Directory Domain Services**
- Accepted all dependency prompts → Installed
- Waited for install to complete (~5 minutes)

### 2. Promoted the server to a Domain Controller
- In Server Manager, clicked the yellow flag notification → "Promote this server to a domain controller"
- Selected: **Add a new forest**
- Root domain name: `lab.local`
- Forest functional level: **Windows Server 2016** (safe default)
- Set a DSRM (Directory Services Restore Mode) password
- Left DNS delegation unchecked (we're the only DNS server)
- Accepted the default paths for AD database, logs, SYSVOL
- Clicked Install → server rebooted automatically
- After reboot, login changed to `LAB\Administrator`

### 3. Created an OU structure
Opened **Active Directory Users and Computers** (ADUC) from Tools menu.

Built this structure under `lab.local`:

```
lab.local
├── IT_Department
│   └── IT_Admins (security group)
├── HR_Department
│   └── HR_Staff (security group)
└── Computers
    └── Workstations
```

Right-click domain → New → Organizational Unit to create each OU.

### 4. Created user accounts
Created the following users inside their respective OUs:

| Name | Username | OU | Group |
|------|----------|----|-------|
| Ermin Kovac | e.kovac | IT_Department | IT_Admins |
| Tarik Mehic | t.mehic | IT_Department | IT_Admins |
| Almin Skeledzija | a.skeledzija | HR_Department | HR_Staff |

Steps per user:
- Right-click OU → New → User
- Filled in first name, last name, logon name
- Set password → unchecked "User must change password at next logon" (lab only)
- After creating, right-click user → Add to group → typed group name

### 5. Verified with PowerShell
Ran these commands to confirm everything was set up correctly:

```powershell
# List all OUs
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# List all users
Get-ADUser -Filter * | Select-Object Name, SamAccountName, Enabled

# List members of IT_Admins group
Get-ADGroupMember -Identity "IT_Admins" | Select-Object Name
```

---

## Issues I ran into

**"DNS delegation could not be created" warning during DC promotion**
- This warning appeared but didn't stop the install
- It just means there's no parent DNS zone to delegate to — normal for a
  brand new lab domain that isn't part of a larger DNS hierarchy
- Safely ignored

**Couldn't log in after reboot with my old password**
- After joining a domain, the login format changes
- Had to use `LAB\Administrator` instead of just `Administrator`
- The domain name prefix (`LAB\`) tells Windows which account database to use

---

## What I learned

AD DS is the core of almost every Windows-based corporate environment. Every time
an employee logs into their work PC, opens Outlook, or accesses a file share,
AD is involved — it's the gatekeeper for identity and access.

The connection to **M365 is direct**: Microsoft 365 uses **Entra ID** (formerly
Azure AD) as its cloud identity layer. In hybrid companies, on-premises AD syncs
to Entra ID via a tool called **Azure AD Connect**, so the same users and passwords
work both on-site and in the cloud. Understanding AD DS is the foundation for
understanding how M365 identity works.
