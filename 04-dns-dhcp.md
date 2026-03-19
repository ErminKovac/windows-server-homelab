# 04 — DNS & DHCP

## Goal
Understand how DNS and DHCP work in a Windows Server environment, and why
both are critical infrastructure for any domain.

---

## Part 1 — DNS

### Why DNS matters for Active Directory
DNS is not optional in a Windows Server environment — AD **requires** it.
When a computer tries to find a Domain Controller, it doesn't use an IP address
directly. It sends a DNS query to find the DC. If DNS breaks, users can't log in,
Group Policy stops applying, and the domain effectively stops working.

When I promoted `WS-LAB-01` to a Domain Controller, Windows automatically
installed a DNS server and created a forward lookup zone for `lab.local`.

### What I explored in DNS Manager

Opened **DNS Manager** from Server Manager → Tools.

**Forward Lookup Zone — `lab.local`**  
This zone resolves names → IP addresses (e.g. `WS-LAB-01.lab.local` → `192.168.1.10`)

Records I found automatically created by AD:
- **A record** for `WS-LAB-01` pointing to `192.168.1.10`
- **_msdcs** subfolder — contains SRV records that computers use to locate DCs
- **SRV records** — critical records that advertise services like Kerberos and LDAP

**Reverse Lookup Zone**  
Resolves IP addresses → names (opposite direction). Created one manually:
1. Right-click Reverse Lookup Zones → New Zone
2. Chose the network `192.168.1.0/24`
3. AD-integrated, replication to all DCs in domain
4. Right-click `WS-LAB-01` A record in forward zone → "Create associated PTR record"

### Testing DNS
```powershell
# Basic DNS resolution test
nslookup WS-LAB-01.lab.local

# Look up the DC by its SRV record (how domain computers find a DC)
nslookup -type=SRV _ldap._tcp.lab.local

# Test from PowerShell
Resolve-DnsName WS-LAB-01.lab.local
```

All returned the correct IP — DNS was working correctly.

---

## Part 2 — DHCP

### What DHCP does
When a device joins a network, it needs an IP address. DHCP (Dynamic Host
Configuration Protocol) hands out IP addresses automatically so you don't
have to configure each device manually. It also tells devices the default
gateway (router), DNS server, and subnet mask.

### Installing the DHCP role
1. Server Manager → Add Roles and Features → DHCP Server
2. After install: yellow flag → "Complete DHCP configuration" → Authorized the
   DHCP server in AD (required so rogue DHCP servers can't hand out wrong addresses)

### Creating a DHCP Scope
A "scope" is the pool of IP addresses DHCP is allowed to hand out.

1. Opened **DHCP Manager** → Server Manager → Tools → DHCP
2. Expanded `WS-LAB-01` → IPv4 → right-click → New Scope
3. Configured:

| Setting | Value |
|---------|-------|
| Scope name | Lab Network |
| Start IP | 192.168.1.100 |
| End IP | 192.168.1.200 |
| Subnet mask | 255.255.255.0 |
| Default gateway | 192.168.1.1 |
| DNS server | 192.168.1.10 (WS-LAB-01) |
| Lease duration | 8 days |

4. Activated the scope
5. Set an **exclusion** for `192.168.1.1 - 192.168.1.99` to reserve that range
   for servers and printers with static IPs

### DHCP Reservations
Created a reservation so a specific device always gets the same IP:
- Right-click Reservations → New Reservation
- Set a friendly name, the device's MAC address, and the IP to always assign
- Useful for printers, servers, or any device that needs a consistent address

### Testing DHCP
```powershell
# Release current IP lease
ipconfig /release

# Request a new IP from DHCP
ipconfig /renew

# View current IP configuration
ipconfig /all

# View all active DHCP leases on the server
Get-DhcpServerv4Lease -ScopeId 192.168.1.0
```

---

## Issues I ran into

**DHCP server showed as "unauthorized" (red arrow)**
- DHCP servers in an AD domain must be authorized to prevent rogue servers
- Fixed by right-clicking the server in DHCP Manager → Authorize
- After a few seconds, restarted the DHCP service and the red arrow turned green

---

## What I learned

DNS and DHCP are foundational network services that run silently in the background
of every corporate network. As a sysadmin you're not often configuring them from
scratch, but you need to know how to read them, troubleshoot them, and understand
what breaks when they go wrong — because when DNS or DHCP fails, the whole
network effectively stops working and everyone comes to the sysadmin.
