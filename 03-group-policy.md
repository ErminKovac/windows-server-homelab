# 03 — Group Policy

## Goal
Create Group Policy Objects (GPOs) and link them to OUs to enforce settings
across users and computers automatically — without touching each machine individually.

---

## Key concepts

| Term | What it means |
|------|--------------|
| **GPO (Group Policy Object)** | A set of settings that gets applied to users or computers in an OU |
| **GPMC** | Group Policy Management Console — the tool used to create and manage GPOs |
| **Linked GPO** | A GPO attached to an OU so it applies to everything inside that OU |
| **gpupdate /force** | A command that immediately applies pending Group Policy changes |

---

## GPOs I created

### GPO 1 — Desktop wallpaper for IT department
**Purpose:** Set a standard desktop background for all IT users  
**Linked to:** `IT_Department` OU

Steps:
1. Opened **Group Policy Management** from Server Manager → Tools
2. Right-click `IT_Department` OU → "Create a GPO in this domain and link it here"
3. Named it: `IT - Desktop Wallpaper`
4. Right-click the new GPO → Edit → opened Group Policy Management Editor
5. Navigated to:
   `User Configuration → Policies → Administrative Templates → Desktop → Desktop`
6. Double-clicked **"Desktop Wallpaper"** → Enabled
7. Set wallpaper path to: `C:\Windows\Web\Wallpaper\Windows\img0.jpg`
8. Wallpaper style: Fill → OK

### GPO 2 — Disable USB storage for HR department
**Purpose:** Prevent HR users from copying files to USB drives (data security)  
**Linked to:** `HR_Department` OU

Steps:
1. Right-click `HR_Department` OU → "Create a GPO in this domain and link it here"
2. Named it: `HR - Disable USB Storage`
3. Edit → navigated to:
   `Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access`
4. Double-clicked **"All Removable Storage classes: Deny all access"** → Enabled → OK

### GPO 3 — Password policy (domain-wide)
**Purpose:** Enforce minimum password requirements for all accounts  
**Linked to:** Domain root (`lab.local`)

Steps:
1. Right-click `lab.local` → Create a GPO → named it `Default - Password Policy`
2. Edit → navigated to:
   `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`
3. Configured:
   - Minimum password length: 10 characters
   - Password complexity requirements: Enabled
   - Maximum password age: 90 days
   - Enforce password history: 5 passwords

---

## Testing that GPOs applied

```powershell
# Force an immediate Group Policy refresh
gpupdate /force

# View which GPOs are currently applied to this machine
gpresult /r

# View a detailed HTML report of applied GPOs
gpresult /h C:\GPOReport.html
```

Opened `C:\GPOReport.html` in a browser — showed all three GPOs listed
under "Applied Group Policy Objects". Confirmed they were applying correctly.

---

## What I learned

Group Policy is one of the most powerful tools in a sysadmin's toolkit. Instead of
configuring 200 laptops one by one, you create one GPO and it pushes to every
machine in the OU automatically — and enforces itself again at every startup or login.

In real-world M365 environments, **Intune** plays a similar role to GPO but for
cloud-managed devices. The logic is the same: you define a policy, assign it to
a group of users or devices, and it enforces itself. Understanding GPO makes
understanding Intune policies much easier.
