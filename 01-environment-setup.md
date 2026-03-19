# 01 — Environment Setup

## Goal
Get a working Windows Server 2022 virtual machine running on my personal PC
at zero cost, so I can practice administration in a safe, isolated environment.

---

## My hardware

| Component | Details |
|-----------|---------|
| CPU | AMD Ryzen 5 3600 (6-core) |
| RAM | 4 GB allocated to VM |
| Motherboard | MSI |
| Host OS | Windows 11 |

---

## What I used

- **VirtualBox 7** — free hypervisor from virtualbox.org
- **Windows Server 2022 Evaluation ISO**  free 180-day trial from microsoft.com/evalcenter
  (no credit card required, just a Microsoft account)

---

## Steps I followed

### 1. Downloaded VirtualBox
- Went to virtualbox.org → Downloads → Windows hosts
- Installed with default options

### 2. Downloaded the Windows Server 2022 ISO
- Went to microsoft.com/evalcenter
- Selected "Windows Server 2022"
- Registered with a free Microsoft account
- Downloaded the ISO file (~5 GB)

### 3. Created the Virtual Machine
- In VirtualBox: New → Named it `WS-LAB`
- Type: Microsoft Windows / Version: Windows Server 2022 (64-bit)
- Unchecked "Proceed with Unattended Installation" — important so you can
  manually choose the Desktop Experience edition during setup
- RAM: 4096 MB (4 GB)
- Processors: 2 vCPUs
- Virtual hard disk: 50 GB, dynamically allocated
- Attached the ISO as the boot disk

### 4. Installed Windows Server 2022
- Boot from ISO → selected **Windows Server 2022 Standard Evaluation (Desktop Experience)**
  — the Desktop Experience option gives you a full GUI, important for learning
- Selected: Custom install (not Upgrade — this is a fresh VM with no existing OS)
- Used the full 50 GB disk
- Set a strong local Administrator password

### 5. Installed VirtualBox Guest Additions
- In VirtualBox menu: Devices → Insert Guest Additions CD Image
- Inside the VM: opened File Explorer → CD Drive → ran VBoxWindowsAdditions.exe
- Installed with defaults → rebooted
- After reboot: screen resolution adjusted properly and mouse integration worked

### 6. Renamed the computer
- Right-click Start → System → Rename this PC
- Changed from the default random name to: `WS-LAB`
- Restarted the VM

### 7. Set a static IP address
- Right-click Start → Network Connections → Change adapter options
- Right-click network adapter → Properties
- Double-clicked Internet Protocol Version 4 (TCP/IPv4)
- Selected "Use the following IP address" and configured:

| Field | Value |
|-------|-------|
| IP address | `192.168.1.10` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.1.1` |
| Preferred DNS | `127.0.0.1` |

- DNS is set to 127.0.0.1 (the server itself) because Active Directory requires
  the server to be its own DNS server — this must be set before installing AD DS

---

## Issues I ran into

**AMD-V virtualization was disabled in BIOS (VERR_SVM_DISABLED)**
- VirtualBox threw this error and the VM aborted on the very first start
- The error means the CPU supports virtualization but it was turned off in BIOS
- Fixed by:
  1. Restarting the PC and pressing Delete to enter BIOS (MSI motherboard)
  2. Going to Advanced → CPU Configuration
  3. Finding SVM Mode (AMD's name for virtualization) and setting it to Enabled
  4. Pressing F10 to save and exit
- After rebooting into Windows, the VM started without any errors
- Lesson: virtualization must be enabled in BIOS before any hypervisor
  (VirtualBox, Hyper-V, VMware) will work. On AMD CPUs it's called SVM Mode,
  on Intel CPUs it's called Intel VT-x.

**Sending Ctrl+Alt+Delete to the VM**
- Pressing Ctrl+Alt+Delete on the keyboard goes to the host PC, not the VM
- Fixed by using the VirtualBox menu: Input → Keyboard → Insert Ctrl+Alt+Delete
- This sends the key combination specifically to the VM

**Taking screenshots of the VM**
- Print Screen captured the host PC desktop instead of the VM
- Fixed by using VirtualBox menu: View → Take Screenshot
- Saves directly as a PNG file on the host PC

---

## What I learned

A VM is essentially a computer inside your computer. VirtualBox creates an isolated
environment so you can safely break things and rebuild without affecting your real PC.
The "Evaluation" version of Windows Server is the full product — Microsoft gives it
free for 180 days specifically so IT people can learn and test. In a real job, the
company pays for licenses, but the skills transfer completely.

Setting a static IP before installing Active Directory is critical. AD needs a
stable, predictable IP address — if the server's IP changes, DNS breaks and the
whole domain stops working.
