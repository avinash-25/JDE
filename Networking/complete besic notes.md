
# Networking Basics for JDE CNC Administrators

> A practical guide covering foundational networking concepts mapped directly to JDE EnterpriseOne administration.
> Every topic includes real-world CNC scenarios, commands, and troubleshooting examples.

---

## Table of Contents

1. [OSI Model & TCP/IP](#1-osi-model--tcpip)
2. [IP Addressing & Subnets](#2-ip-addressing--subnets)
3. [DNS & Hostname Resolution](#3-dns--hostname-resolution)
4. [Ports & Protocols](#4-ports--protocols)
5. [Firewalls & ACLs](#5-firewalls--acls)
6. [DHCP vs Static IP](#6-dhcp-vs-static-ip)

---

## 1. OSI Model & TCP/IP

### What It Is

The OSI (Open Systems Interconnection) model breaks network communication into **7 layers**. TCP/IP condenses these into **4 practical layers** that real networks actually use. As a CNC admin, you use this model as a **diagnostic framework** — when something breaks, you work bottom-up to find where it failed.

```
| OSI Layer      | TCP/IP Layer   | What Happens Here                       |
| -------------- | -------------- | --------------------------------------- |
| 7 Application  | Application    | HTTP, HTTPS, DNS, LDAP — JDE UI, APIs   |
| 6 Presentation | Application    | SSL/TLS encryption — JDE certificates   |
| 5 Session      | Application    | JDENET sessions between servers         |
| 4 Transport    | Transport      | TCP connections — port 6009, 1521, 8080 |
| 3 Network      | Internet       | IP addresses, routing between servers   |
| 2 Data Link    | Network Access | Ethernet, MAC addresses, VLANs          |
| 1 Physical     | Network Access | Cables, NICs, switches                  |
```

### The JDE Component Stack

```
User Browser
    |  HTTPS (Layer 7)
    v
JDE HTML / Web Server (WebLogic/Tomcat)   ← Layer 7: HTTP/HTTPS on port 8080/443
    |  JDENET TCP (Layer 4/5)
    v
JDE Enterprise Server                      ← Layer 4: TCP port 6009–6017
    |  JDBC TCP (Layer 4)
    v
Oracle / SQL Server Database               ← Layer 4: TCP port 1521 (Oracle)
```

### Real-Time Example: User Cannot Log In to JDE

A user calls saying they cannot open the JDE login page. Here is how you diagnose using the OSI model — bottom-up:

**Step 1 — Layer 3 (Network): Can the web server even be reached?**

```bash
# From your machine, ping the JDE web server
ping jde-web-server01

# Expected: Reply from 10.10.1.50
# Bad: Request timed out  →  routing or firewall blocking ICMP
```

**Step 2 — Layer 4 (Transport): Is the port open?**

```bash
# Test if port 8080 is accepting connections
telnet jde-web-server01 8080

# Windows alternative
Test-NetConnection -ComputerName jde-web-server01 -Port 8080

# Expected: Connected to jde-web-server01
# Bad: Connection refused  →  WebLogic not started, or firewall blocking port
```

**Step 3 — Layer 6 (Presentation): Is the SSL certificate valid?**

```bash
# On the JDE web server, check certificate in the keystore
keytool -list -v -keystore /u01/jde/keystore.jks -storepass changeit | grep "Valid"

# Look for: Valid from: Mon Jan 01 ... until: Wed Dec 31 ...
# Bad: Expired date  →  renew and reimport the certificate
```

**Step 4 — Layer 7 (Application): Is WebLogic actually serving pages?**

```bash
# Check WebLogic server log on the JDE web server
tail -100 /u01/Oracle/Middleware/user_projects/domains/JDE_E1/servers/JDE_Server1/logs/JDE_Server1.log

# Look for: <Server started in RUNNING mode>
# Bad: OutOfMemoryError, SocketException  →  JVM heap issue or JDENET misconfiguration
```

> **CNC Rule:** Never jump straight to the JDE application log. If the network path is broken at Layer 3 or 4, application-level fixes will change nothing.

---

## 2. IP Addressing & Subnets

### What It Is

Every device on a network has an **IP address** — a unique numeric label. In JDE environments, all servers (web, enterprise, database, deployment) are identified by their IP addresses. Subnets define which machines are on the same local network segment.

### IPv4 Basics

```
IP Address:   10.10.1.50
              |  |  | |
              |  |  | └── Host part (which machine on this subnet)
              |  |  └──── Subnet
              |  └─────── Network
              └────────── Network class / range

Subnet Mask:  255.255.255.0   (also written as /24)
              means: first 24 bits = network, last 8 bits = host

Network:      10.10.1.0/24   → allows hosts 10.10.1.1 to 10.10.1.254
Broadcast:    10.10.1.255    → sends to all hosts on this subnet
```

### Private IP Ranges (Always Used for JDE Servers)

```
| Range                         | CIDR | Common Use in JDE               |
| ----------------------------- | ---- | ------------------------------- |
| 10.0.0.0 – 10.255.255.255     | /8   | Large enterprise networks       |
| 172.16.0.0 – 172.31.255.255   | /12  | Medium-sized enterprise         |
| 192.168.0.0 – 192.168.255.255 | /16  | Small office / lab environments |
```

### Typical JDE Server IP Layout

```
Subnet: 10.10.1.0/24  (Internal Application Zone)
-----------------------------------------------------
10.10.1.10   →  JDE Deployment Server (DS)
10.10.1.20   →  JDE Enterprise Server (ES)
10.10.1.21   →  JDE Enterprise Server 2 (ES - failover)
10.10.1.30   →  Oracle Database Server (DB)

Subnet: 10.10.2.0/24  (DMZ — Web Tier)
---------------------------------------------
10.10.2.10   →  JDE HTML Web Server 1
10.10.2.11   →  JDE HTML Web Server 2
10.10.2.5    →  Load Balancer VIP (Virtual IP)

Subnet: 10.10.3.0/24  (Management Zone)
---------------------------------------------
10.10.3.50   →  CNC Admin workstation
10.10.3.51   →  Monitoring server
```

### Real-Time Example: Adding a New Enterprise Server

Your company is adding a second Enterprise Server for load balancing. You need to verify IP settings before registering it in JDE.

**Step 1 — Check the IP configuration on the new server**

```bash
# Linux (RHEL/OEL — common for JDE Enterprise Servers)
ip addr show
# or older command:
ifconfig

# Expected output:
# inet 10.10.1.21/24 brd 10.10.1.255 scope global ens192

# Windows Server
ipconfig /all
# Look for: IPv4 Address: 10.10.1.21, Subnet Mask: 255.255.255.0
```

**Step 2 — Confirm it can reach the database server (different subnet requires routing)**

```bash
ping 10.10.1.30        # same subnet — should always work
ping 10.10.2.10        # different subnet — needs router/firewall rule
```

**Step 3 — Calculate if two servers are on the same subnet**

```
Server A: 10.10.1.20 /24
Server B: 10.10.1.21 /24
Same network? 10.10.1.x — YES, direct communication, no router needed.

Server A: 10.10.1.20 /24
Server C: 10.10.2.10 /24
Same network? 10.10.1.x vs 10.10.2.x — NO, traffic must go through a router/firewall.
```

**Step 4 — Register the new server IP in jde.ini**

```ini
; On the HTML Server's jde.ini, add/update the Enterprise Server entry
[JDENET_KERNEL_DEF2]
kType=2
kConnect=1
kPortNumber=6010
; Reference the server by hostname (DNS resolves to IP)
; Make sure DNS has an A record: jde-es02 → 10.10.1.21
```

> **CNC Rule:** JDE servers must always use **static IP addresses**. If the Enterprise Server's IP changes (e.g., DHCP lease expires), all jde.ini files pointing to it will break. See Section 6 for DHCP vs static.

---

## 3. DNS & Hostname Resolution

### What It Is

DNS (Domain Name System) translates human-readable hostnames (like `jde-enterprise01`) into IP addresses (like `10.10.1.20`). JDE relies on DNS heavily — all server-to-server communication in jde.ini uses **hostnames**, not IP addresses directly.

### How Resolution Works

```
JDE HTML Server needs to connect to "jde-enterprise01"
          |
          ▼
1. Check local hosts file first
   (C:\Windows\System32\drivers\etc\hosts  or  /etc/hosts)
          |
          ▼ (not found in hosts file)
2. Ask DNS server (configured in NIC settings)
   DNS Server: 10.10.0.10
          |
          ▼
3. DNS responds: jde-enterprise01 → 10.10.1.20
          |
          ▼
4. HTML Server connects to 10.10.1.20 on port 6009
```

### Key DNS Record Types for JDE

```
| Record Type | Purpose                        | JDE Example                                         |
| ----------- | ------------------------------ | --------------------------------------------------- |
| A           | Hostname → IPv4 address        | jde-enterprise01 → 10.10.1.20                       |
| CNAME       | Alias → another hostname       | jde-es → jde-enterprise01 (alias for load balancer) |
| PTR         | IP → Hostname (reverse lookup) | 10.10.1.20 → jde-enterprise01                       |
```

### Real-Time Example: JDENET Connection Failure After Server Rename

The enterprise server was renamed from `jdeapp01` to `jde-enterprise01` during a server migration. After the rename, the HTML Server cannot connect to the Enterprise Server.

**Step 1 — Check what hostname the HTML Server is using (from jde.ini)**

```ini
; On the HTML Server, open:
; C:\JDEdwards\E920\ini\JDE.INI  (Windows)
; /JDEdwards/E920/ini/jde.ini    (Linux)

[JDENET_KERNEL_DEF1]
kType=2
kConnect=1
kPortNumber=6009
; This hostname must resolve correctly:
; (hostname is set elsewhere in the config — check the server name entries)
```

**Step 2 — Test DNS resolution from the HTML Server**

```bash
# Windows
nslookup jde-enterprise01
# Expected: Name: jde-enterprise01, Address: 10.10.1.20
# Bad: can't find jde-enterprise01: Non-existent domain

# Linux
nslookup jde-enterprise01
# or
dig jde-enterprise01 A

# Also test reverse lookup
nslookup 10.10.1.20
# Expected: name = jde-enterprise01.yourdomain.com
```

**Step 3 — If DNS is wrong, test with a hosts file override**

```bash
# Add a temporary entry to the hosts file to bypass DNS
# Windows: C:\Windows\System32\drivers\etc\hosts
# Linux:   /etc/hosts

10.10.1.20   jde-enterprise01   jde-enterprise01.company.com

# Save file, then test:
nslookup jde-enterprise01
# Should now resolve from hosts file
```

**Step 4 — Verify the Enterprise Server's own hostname matches**

```bash
# On the Enterprise Server itself
hostname
# Must return: jde-enterprise01
# If it returns the old name 'jdeapp01', the OS hostname was not updated

# Linux — update hostname permanently
hostnamectl set-hostname jde-enterprise01

# Windows
Rename-Computer -NewName "jde-enterprise01" -Restart
```

**Step 5 — Check the Deployment Server's OCM (Object Configuration Manager)**

```
In JDE Web Client → Server Administration Workbench (SAW)
→ Server Map → Verify server name matches the new hostname exactly
→ OCM mappings referencing old hostname must be updated
```

> **CNC Rule:** In JDE, **hostnames must match everywhere**: jde.ini, OCM/Server Map in the database, the OS hostname, and DNS. A mismatch at any one of these four places will break connectivity.

---

## 4. Ports & Protocols

### What It Is

A **port** is a numbered logical endpoint on a server. When a service starts, it "listens" on a specific port. JDE has many components, and each uses specific ports for communication. A firewall that blocks even one required port will silently break a specific JDE function.

### TCP vs UDP

```
TCP (Transmission Control Protocol)
- Connection-oriented: establishes a connection before sending data
- Guarantees delivery, order, and error checking
- Used by: ALL JDE inter-component communication
- Example: HTML Server → Enterprise Server via JDENET (TCP 6009)

UDP (User Datagram Protocol)
- Connectionless: fire-and-forget
- No guarantee of delivery
- Used by: DNS queries (UDP 53), some monitoring tools
- JDE itself does not use UDP for core communication
```

### Complete JDE Port Reference

```
| Component / Service         | Port(s)     | Protocol | Direction                      |
| --------------------------- | ----------- | -------- | ------------------------------ |
| JDE HTML/Web Server (HTTP)  | 8080        | TCP      | Users → Web Server             |
| JDE HTML/Web Server (HTTPS) | 443 / 8443  | TCP      | Users → Web Server             |
| WebLogic Admin Console      | 7001        | TCP      | CNC Admin → Web Server         |
| JDE AIS Server              | 8080 / 8443 | TCP      | Mobile/API → AIS Server        |
| JDE Orchestrator Framework  | 8080 / 8443 | TCP      | Integrations → Orchestrator    |
| JDENET (Enterprise Server)  | 6009        | TCP      | Web Server → Enterprise Server |
| JDE Kernel ports            | 6010–6017   | TCP      | Web Server → Enterprise Server |
| Oracle Database             | 1521        | TCP      | Enterprise Server → DB         |
| SQL Server Database         | 1433        | TCP      | Enterprise Server → DB         |
| JDE Deployment Server RPC   | 445 / 139   | TCP      | Deployment operations          |
| LDAP (Active Directory)     | 389         | TCP      | JDE → AD (authentication)      |
| LDAPS (AD over SSL)         | 636         | TCP      | JDE → AD (secure auth)         |
| SSH (Linux server admin)    | 22          | TCP      | CNC Admin → Linux servers      |
| RDP (Windows server admin)  | 3389        | TCP      | CNC Admin → Windows servers    |
```

### Real-Time Example: Business Function (BSFN) Calls Hanging

A custom orchestration is calling a Business Function on the Enterprise Server. The calls hang for 60 seconds and then time out. Users report BSFNs are slow or failing.

**Step 1 — Check if the JDENET port is open and responsive**

```bash
# From the HTML/AIS Server, test connectivity to Enterprise Server JDENET port
telnet jde-enterprise01 6009
# Expected: blank screen (connected, waiting)
# Bad: 'Could not open connection' after timeout  →  port blocked or service down

# More informative test on Windows:
Test-NetConnection -ComputerName jde-enterprise01 -Port 6009
# TcpTestSucceeded: True  ← good
# TcpTestSucceeded: False ← port blocked or service not listening
```

**Step 2 — Check listening ports on the Enterprise Server itself**

```bash
# Windows Enterprise Server
netstat -an | findstr "6009"
# Expected: TCP  0.0.0.0:6009  0.0.0.0:0  LISTENING

# Linux Enterprise Server
netstat -tlnp | grep 6009
# or
ss -tlnp | grep 6009
# Expected: LISTEN  0  128  0.0.0.0:6009  0.0.0.0:*

# If port 6009 is NOT listed as LISTENING — the JDE kernel is not running
```

**Step 3 — Check active JDENET connections (ESTABLISHED = connected clients)**

```bash
# Windows
netstat -an | findstr "6009" | findstr "ESTABLISHED"

# Linux
netstat -an | grep 6009 | grep ESTABLISHED

# If you see ESTABLISHED connections, the service is up
# Count them — if MaxConnections is hit, new requests queue up → slowness
```

**Step 4 — Check kernel port range in jde.ini**

```ini
; On the Enterprise Server, open jde.ini
[JDENET]
serviceNameListen=6009
serviceNameConnect=6009

[JDENET_KERNEL_DEF1]
kType=2
kConnect=1
kPortNumber=6009      ; primary kernel port

; If you have multiple kernels, they use 6010, 6011, etc.
; Ensure ALL ports in this range are open in the firewall
```

> **CNC Rule:** When you open a ticket with the network/firewall team, always give them the **exact source IP, destination IP, destination port, and protocol** (TCP/UDP). Example: "Please open TCP port 6009 from subnet 10.10.2.0/24 (web DMZ) to 10.10.1.20 (Enterprise Server)."

---

## 5. Firewalls & ACLs

### What It Is

A **firewall** is a security system that controls which network traffic is allowed to pass between network segments. Rules are evaluated in order — first match wins. An **ACL (Access Control List)** is the list of rules defining what is allowed or denied. In JDE environments, firewalls sit between the DMZ (web tier) and the internal network (application and database tiers).

### How Firewall Rules Work

```
Rule evaluation (top to bottom, first match wins):

| Rule # | Source IP    | Destination IP | Port | Protocol | Action                               |
| ------ | ------------ | -------------- | ---- | -------- | ------------------------------------ |
| 1      | 10.10.2.0/24 | 10.10.1.20     | 6009 | TCP      | ALLOW   ← Web→ES JDENET              |
| 2      | 10.10.2.0/24 | 10.10.1.20     | 6010 | TCP      | ALLOW   ← Web→ES kernel              |
| 3      | 10.10.2.0/24 | 10.10.1.30     | 1521 | TCP      | DENY    ← Web cannot hit DB directly |
| 4      | 10.10.1.20   | 10.10.1.30     | 1521 | TCP      | ALLOW   ← ES→DB Oracle               |
| 5      | ANY          | ANY            | ANY  | ANY      | DENY    ← Default deny all           |
```

### JDE Three-Tier Firewall Architecture

```
Internet / Users
      |
      | HTTPS 443 / HTTP 8080
      ▼
 ┌─────────────────────────────────┐
 │         DMZ (Web Zone)          │
 │   JDE HTML Web Server           │
 │   JDE AIS Server                │
 │   Load Balancer                 │
 └─────────────────────────────────┘
      |
      | JDENET TCP 6009–6017 (ALLOWED through firewall)
      | LDAP TCP 389 (if AD is in internal zone)
      ▼
 ┌─────────────────────────────────┐
 │      Internal (App Zone)        │
 │   JDE Enterprise Server         │
 │   JDE Deployment Server         │
 └─────────────────────────────────┘
      |
      | Oracle JDBC TCP 1521 (ALLOWED)
      ▼
 ┌─────────────────────────────────┐
 │      Restricted (DB Zone)       │
 │   Oracle / SQL Server           │
 └─────────────────────────────────┘

KEY RULE: Web Zone servers can NEVER directly reach the DB Zone.
          Only the Enterprise Server (App Zone) can talk to the database.
```

### Real-Time Example: New JDE Environment — Firewall Change Request

You are deploying a new JDE EnterpriseOne 9.2 environment. You need to submit a firewall change request to the network security team.

**Firewall Change Request (FCR) — what to include:**

```
FCR Title: JDE EnterpriseOne 9.2 Production Deployment — Firewall Rules

Rules Required:
---------------------------------------------------------------------------
Rule 1: Web → Enterprise Server (JDENET)
  Source:      10.10.2.0/24  (DMZ Web Zone)
  Destination: 10.10.1.20    (jde-enterprise01)
  Port:        6009–6017
  Protocol:    TCP
  Direction:   Outbound from DMZ / Inbound to App Zone
  Purpose:     JDENET communication between HTML server and Enterprise Server

Rule 2: Enterprise Server → Oracle Database (JDBC)
  Source:      10.10.1.20    (jde-enterprise01)
  Destination: 10.10.1.30    (jde-oracle-db01)
  Port:        1521
  Protocol:    TCP
  Direction:   Outbound from App Zone / Inbound to DB Zone
  Purpose:     JDBC database connectivity for JDE business functions

Rule 3: CNC Admin → All JDE Servers (Administration)
  Source:      10.10.3.50    (cnc-admin-workstation)
  Destination: 10.10.1.20, 10.10.1.10, 10.10.2.10
  Ports:       22 (SSH), 3389 (RDP), 7001 (WebLogic console)
  Protocol:    TCP
  Purpose:     CNC administrative access to all servers

Rule 4: JDE → Active Directory (Authentication)
  Source:      10.10.1.20    (jde-enterprise01)
  Destination: 10.10.0.5     (ad-server01)
  Port:        389 (LDAP) or 636 (LDAPS)
  Protocol:    TCP
  Purpose:     JDE user authentication via Active Directory
---------------------------------------------------------------------------
```

**Testing firewall rules after they are applied:**

```bash
# Test each rule from the correct source machine

# Rule 1 — From the web server, test JDENET port:
Test-NetConnection -ComputerName 10.10.1.20 -Port 6009

# Rule 2 — From the Enterprise Server, test Oracle port:
telnet 10.10.1.30 1521

# Rule 4 — From the Enterprise Server, test LDAP:
Test-NetConnection -ComputerName 10.10.0.5 -Port 389
```

**Diagnosing a blocked port (firewall rule missing):**

```bash
# Symptom: JDE login fails with "Server is unavailable"
# Test from HTML Server:
Test-NetConnection -ComputerName jde-enterprise01 -Port 6009

# Output:
# ComputerName     : jde-enterprise01
# TcpTestSucceeded : False   ← firewall is blocking this port

# Escalate to network team:
# "TCP port 6009 from 10.10.2.10 to 10.10.1.20 is being blocked.
#  Please check ACL rule between DMZ and App Zone."
```

> **CNC Rule:** Always test firewall rules **from the actual source server**, not from your admin workstation. A rule that allows 10.10.2.10 → 10.10.1.20 on port 6009 does not help you if you test it from 10.10.3.50 (your workstation).

---

## 6. DHCP vs Static IP

### What It Is

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses to devices from a pool, with a time-limited "lease." When the lease expires, the device may get a different IP. **Static IP** means the IP address is manually configured and never changes.

### Why JDE Servers Must Always Use Static IPs

```
Scenario: JDE Enterprise Server uses DHCP

Day 1:   Server gets IP  10.10.1.20 (DHCP lease)
         jde.ini on all HTML servers points to: jde-enterprise01 (DNS → 10.10.1.20)
         Everything works.

Day 30:  DHCP lease expires. Server reboots.
         Server gets new IP: 10.10.1.45 (new DHCP lease)
         DNS still points jde-enterprise01 → 10.10.1.20 (old IP)
         10.10.1.20 is now a DIFFERENT server or empty.

Result:  ALL JDE HTML servers cannot connect to Enterprise Server.
         JDENET connections fail. All users locked out.
         Firewall rules (written for 10.10.1.20) no longer apply.
         You chase a phantom problem for hours.
```

### Static IP Assignment — JDE Server Checklist

Every JDE server needs these configured statically:

```
1. IP Address       →  e.g. 10.10.1.20
2. Subnet Mask      →  e.g. 255.255.255.0
3. Default Gateway  →  e.g. 10.10.1.1  (router that connects to other subnets)
4. DNS Server(s)    →  e.g. 10.10.0.10 (primary), 10.10.0.11 (secondary)
5. Hostname         →  e.g. jde-enterprise01 (must match DNS A record)
```

### Real-Time Example: Configuring Static IP on a New JDE Enterprise Server

**Linux (RHEL/Oracle Enterprise Linux) — most common for JDE Enterprise Server:**

```bash
# Step 1 — Find the network interface name
ip addr show
# Look for interface like: ens192, eth0, enp3s0

# Step 2 — Edit the network configuration file
# RHEL 7/OEL 7:
vi /etc/sysconfig/network-scripts/ifcfg-ens192

# Set the following:
BOOTPROTO=none        # 'none' or 'static' — NOT 'dhcp'
IPADDR=10.10.1.20
NETMASK=255.255.255.0
GATEWAY=10.10.1.1
DNS1=10.10.0.10
DNS2=10.10.0.11
ONBOOT=yes
NAME=ens192

# Step 3 — Restart networking
systemctl restart network
# RHEL 8+:
nmcli connection reload
nmcli connection up ens192

# Step 4 — Verify
ip addr show ens192
# Expected: inet 10.10.1.20/24

# Step 5 — Test gateway reachability
ping 10.10.1.1
# Expected: Reply from 10.10.1.1
```

**Windows Server — common for JDE Deployment Server and HTML Server:**

```powershell
# PowerShell — set static IP on a Windows JDE server

# Step 1 — Find interface index
Get-NetAdapter
# Note the InterfaceIndex, e.g. 6

# Step 2 — Remove existing DHCP assignment
Remove-NetIPAddress -InterfaceIndex 6 -Confirm:$false
Remove-NetRoute -InterfaceIndex 6 -Confirm:$false

# Step 3 — Set static IP
New-NetIPAddress `
  -InterfaceIndex 6 `
  -IPAddress 10.10.1.10 `
  -PrefixLength 24 `
  -DefaultGateway 10.10.1.1

# Step 4 — Set DNS servers
Set-DnsClientServerAddress `
  -InterfaceIndex 6 `
  -ServerAddresses 10.10.0.10, 10.10.0.11

# Step 5 — Verify
ipconfig /all
# Look for: IPv4 Address: 10.10.1.10, DHCP Enabled: No
```

### DHCP Reservation — an Alternative for Non-Critical Servers

For dev/test JDE environments, a **DHCP reservation** gives the benefits of both: the IP is assigned by DHCP but is always the same, bound to the server's MAC address.

```
DHCP Reservation entry (configured on the DHCP server):
  MAC Address:   00:50:56:AB:12:34   (the server's NIC MAC)
  Reserved IP:   10.10.5.20
  Hostname:      jde-dev-es01

Effect: Every time the server requests an IP via DHCP,
        it always gets 10.10.5.20.
```

> **CNC Rule for production:** Never use DHCP reservations for production JDE servers. Use full static IP configuration. For dev/test, reservations are acceptable but static is still preferred.

---

## Quick Reference: JDE CNC Network Troubleshooting Commands

```bash
# ─── Connectivity Tests ───────────────────────────────────────────────────────

# Ping a JDE server (Layer 3 check)
ping jde-enterprise01
ping 10.10.1.20

# Test a specific port (Layer 4 check) — Windows
Test-NetConnection -ComputerName jde-enterprise01 -Port 6009
telnet jde-enterprise01 6009

# Test a specific port — Linux
nc -zv jde-enterprise01 6009
# or
bash -c "echo > /dev/tcp/jde-enterprise01/6009" && echo "Port open"

# ─── DNS Resolution ───────────────────────────────────────────────────────────

# Forward lookup (hostname → IP)
nslookup jde-enterprise01

# Reverse lookup (IP → hostname)
nslookup 10.10.1.20

# Flush DNS cache (Windows) — if hostname just changed in DNS
ipconfig /flushdns

# ─── Port and Process Checks (run on the JDE server itself) ──────────────────

# What ports are listening? (Windows)
netstat -an | findstr LISTENING
netstat -an | findstr "6009"

# What ports are listening? (Linux)
ss -tlnp
netstat -tlnp | grep java   # JDE processes are Java

# How many ESTABLISHED connections on JDENET port?
netstat -an | findstr "6009" | findstr ESTABLISHED | Measure-Object -Line

# ─── IP and Network Config ───────────────────────────────────────────────────

# Show IP config (Windows)
ipconfig /all

# Show IP config (Linux)
ip addr show
ip route show

# Show route to a destination
tracert jde-enterprise01     # Windows
traceroute jde-enterprise01  # Linux

# ─── JDE Specific ─────────────────────────────────────────────────────────────

# Check jde.ini for server/port config (Windows Deployment Server)
type C:\JDEdwards\E920\ini\JDE.INI | findstr /i "port\|server\|host"

# Check JDE kernel log for connection errors (Linux Enterprise Server)
tail -200 /JDEdwards/E920/logs/jde.log | grep -i "error\|connect\|port"
```

---

## Summary: The 6 Basics and Their CNC Relevance

| Topic                   | Why It Matters for JDE CNC                                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| OSI / TCP/IP            | Gives you a systematic framework to diagnose any JDE connectivity problem from bottom up                     |
| IP Addressing & Subnets | Every server, firewall rule, and jde.ini entry is based on IPs — you must read and plan them confidently     |
| DNS & Hostnames         | JDE uses hostnames everywhere — a wrong DNS record silently breaks inter-component communication             |
| Ports & Protocols       | JDE has a specific port map — knowing it lets you write precise firewall rules and verify services instantly |
| Firewalls & ACLs        | You will submit and verify firewall change requests for every JDE deployment and troubleshoot blocked ports  |
| DHCP vs Static IP       | All production JDE servers need static IPs — misconfigured DHCP causes catastrophic outages                  |

---

*Document version: 1.0 — JDE EnterpriseOne 9.2 focus*
*Topics: Networking Basics for CNC Administration*