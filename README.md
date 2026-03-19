# Windows Server Homelab

Self-directed lab built to learn Windows Server administration from scratch.
Completed as part of preparation for a System Administrator role.

---

## Environment

| Component | Details |
|-----------|---------|
| Host OS | Windows 11 |
| Hypervisor | VirtualBox 7 (free) |
| Guest OS | Windows Server 2022 Evaluation (180-day free trial) |
| VM Name | WS-LAB |
| Domain | lab.local |
| RAM allocated | 4 GB |
| Processors | 2 vCPUs |
| Storage | 50 GB (dynamic) |

---

## What I configured

- [x] Installed Windows Server 2022 with Desktop Experience
- [x] Promoted server to Domain Controller
- [x] Configured Active Directory Domain Services (AD DS)
- [x] Created OU structure, users, and security groups
- [x] Configured DNS (AD-integrated zone)
- [x] Created and linked Group Policy Objects (GPOs)
- [x] Configured DHCP server and scope
- [x] Explored PowerShell for basic administration

---

## Lab notes

| File | Topic |
|------|-------|
| [01-environment-setup.md](01-environment-setup.md) | VirtualBox + Windows Server 2022 install |
| [02-active-directory.md](02-active-directory.md) | AD DS, Domain Controller, OUs, users, groups |
| [03-group-policy.md](03-group-policy.md) | GPOs — creating, linking, and testing |
| [04-dns-dhcp.md](04-dns-dhcp.md) | DNS zones and DHCP scope configuration |
| [05-powershell-basics.md](05-powershell-basics.md) | PowerShell cmdlets for everyday admin tasks |

---

## Key things I learned

- Active Directory is the backbone of identity management in Windows environments.
  Every user, computer, and policy flows through it.
- On-premises AD and **Entra ID (Azure AD)** are closely related — Entra ID is
  essentially AD in the cloud. In hybrid environments they sync via **Azure AD Connect**.
  This is directly relevant to M365 administration.
- AD is completely dependent on DNS. If DNS breaks, the whole domain breaks.
- Group Policy is how you enforce settings at scale across hundreds of machines
  without touching each one individually.
- PowerShell makes repetitive tasks (bulk user creation, querying AD, etc.)
  dramatically faster than using the GUI.

---

## Resources I used

[Microsoft Learn — Windows Server](https://learn.microsoft.com/en-us/windows-server/)
[Windows Server 2022 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
