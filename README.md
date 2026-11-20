# 🌐 Networking Basics for DevOps

A simple, beginner-friendly guide to computer networking for DevOps: what each piece is, why DevOps people need it, and the exact commands you can run to see it with your own eyes.

Every topic follows the same pattern: **what it is**, **why DevOps people care**, and **something small you can try right now**. You don't need any networking background to start. Read it in order, run every command yourself, and finish with the labs at the end.

![Learning path](images/00-roadmap.png)

---

## 📚 Table of Contents

**PART 1: How Networks Actually Work**

1. [Start Here: Why Networking Matters for DevOps](#1-start-here-why-networking-matters-for-devops)
2. [The OSI Model](#2-the-osi-model)
3. [The TCP/IP Model](#3-the-tcpip-model)
4. [Encapsulation: How Data Gets Wrapped](#4-encapsulation-how-data-gets-wrapped)
5. [What Happens When You Open a Website](#5-what-happens-when-you-open-a-website)
6. [IP Addressing](#6-ip-addressing)
7. [Subnet Mask and CIDR](#7-subnet-mask-and-cidr)
8. [Subnetting Step by Step](#8-subnetting-step-by-step)
9. [Public vs Private IP](#9-public-vs-private-ip)
10. [NAT: Network Address Translation](#10-nat-network-address-translation)
11. [DHCP: Getting an IP Automatically](#11-dhcp-getting-an-ip-automatically)
12. [Routing and Gateways](#12-routing-and-gateways) 🆕

**PART 2: Protocols (The Languages)**

13. [TCP vs UDP](#13-tcp-vs-udp)
14. [Ports and Sockets](#14-ports-and-sockets)
15. [DNS](#15-dns)
16. [HTTP and HTTPS](#16-http-and-https)
17. [SSL/TLS Certificates and the HTTPS Handshake](#17-ssltls-certificates-and-the-https-handshake) 🆕
18. [SSH](#18-ssh)
19. [FTP, SFTP and File Transfer](#19-ftp-sftp-and-file-transfer)
20. [SMTP and Email](#20-smtp-and-email)

**PART 3: Network Infrastructure**

21. [Load Balancers (L4 vs L7)](#21-load-balancers)
22. [Proxy and Reverse Proxy](#22-proxy-and-reverse-proxy)
23. [Firewalls and Security Groups](#23-firewalls-and-security-groups)
24. [Linux Firewalls: iptables, ufw, firewalld](#24-linux-firewalls-iptables-ufw-firewalld) 🆕
25. [VPN](#25-vpn)
26. [VPC: Your Private Network in the Cloud](#26-vpc-your-private-network-in-the-cloud) 🆕

**PART 4: Tools and Practice**

27. [The Networking Toolbox](#27-the-networking-toolbox)
28. [Troubleshooting Playbook](#28-troubleshooting-playbook)
29. [Practice Labs](#29-practice-labs)
30. [Cheat Sheet](#30-cheat-sheet)
31. [Interview Questions](#31-interview-questions) 🆕
32. [Reference](#32-reference)

---

# PART 1: How Networks Actually Work

---

## 1. Start Here: Why Networking Matters for DevOps

You can learn Docker, Kubernetes, Terraform and Jenkins, and still be stuck for hours on problems like these:

| What people say | What it actually is |
|-----------------|--------------------|
| "The deploy worked but the site shows 502" | The reverse proxy cannot reach the backend port |
| "It works on my laptop, not on the server" | The app is bound to `127.0.0.1` instead of `0.0.0.0` |
| "The new server cannot download packages" | Private subnet with no NAT gateway |
| "Our domain still points to the old server" | DNS TTL caching |
| "The pipeline times out talking to the database" | Security group does not allow port 3306 from that subnet |
| "SSH just hangs" | Firewall is dropping packets instead of rejecting them |

Every one of those is networking, not DevOps tooling. **Networking is the layer under everything you deploy.**

### 1.1 What you will be able to do after this guide

- Read an IP address and say which network it belongs to
- Split a network into subnets for a cloud VPC
- Explain why a site is slow, broken, or unreachable, **in the right order**
- Open exactly the ports you need and nothing else
- Debug DNS, TLS, proxies and load balancers with `dig`, `curl`, `ss` and `traceroute`

### 1.2 The 6 words you will see everywhere

| Word | Plain meaning |
|------|---------------|
| **Host** | Any machine on the network (laptop, server, phone, container) |
| **Packet** | A small chunk of your data with address labels attached |
| **Protocol** | Agreed rules for a conversation (HTTP, DNS, SSH…) |
| **Port** | A numbered door on a machine, one per program |
| **Gateway** | The way out of your network, normally your router |
| **Latency** | How long a packet takes to get there (measured in ms) |

> 💡 **Tip:** Keep a terminal open while you read. Every section has commands you can run immediately. Networking only makes sense when you watch it happen.

---

## 2. The OSI Model

**What it is:** A 7-layer map that splits "sending data" into small jobs, so people can say *where* a problem is instead of just "the network is broken".

**Why DevOps people care:** It gives you a search order. You never debug randomly - you go bottom-up (is the cable/IP alive?) or top-down (is the app replying?).

![OSI model](images/01-osi-model.png)

### 2.1 The layers, in human words

| # | Layer | Its one job | You meet it as |
|---|-------|-------------|----------------|
| 7 | **Application** | The thing the user actually uses | HTTP, DNS, SSH, SMTP |
| 6 | **Presentation** | Encrypt / compress / format the data | TLS certificates, UTF-8, JPEG |
| 5 | **Session** | Start, keep and close a conversation | Login sessions, cookies |
| 4 | **Transport** | Deliver to the right program, reliably or not | TCP, UDP, **port numbers** |
| 3 | **Network** | Find the best route between networks | **IP addresses**, routers, ICMP |
| 2 | **Data Link** | Move frames between two devices on the same LAN | MAC addresses, switches, ARP |
| 1 | **Physical** | Actual electricity, light or radio | Cables, Wi-Fi, network cards |

**Remember the order (7 → 1):** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

### 2.2 How to use it in real life

```
"The website is down"

Layer 1/2 → Is the machine online at all?          ip link, ip a
Layer 3   → Can I reach the IP?                    ping, traceroute
Layer 4   → Is the port open?                      nc -zv host 443
Layer 7   → Does the app answer correctly?         curl -v https://host
```

> 💡 **Tip:** In the real world people say "a layer 4 load balancer" (works on IP + port) and "a layer 7 load balancer" (reads HTTP). Now you know exactly what that means.

> ⚠️ **Warning:** Layers 5 and 6 barely exist in practice. Real traffic uses the TCP/IP model in the next section. Learn OSI for the vocabulary, use TCP/IP for the work.

---

## 3. The TCP/IP Model

**What it is:** The 4-layer model the internet is actually built on. Same idea as OSI, fewer boxes.

![OSI vs TCP/IP](images/02-osi-vs-tcpip.png)

| TCP/IP layer | Swallows OSI layers | Examples |
|--------------|--------------------|----------|
| **Application** | 7 + 6 + 5 | HTTP, HTTPS, DNS, SSH, SMTP, FTP |
| **Transport** | 4 | TCP, UDP |
| **Internet** | 3 | IP, ICMP, routing |
| **Network Access** (Link) | 2 + 1 | Ethernet, Wi-Fi, ARP, cables |

### 3.1 Why two models exist

- **OSI** is the teaching model and the shared vocabulary ("that's a layer 3 problem").
- **TCP/IP** is the working model - it describes the protocols that really run.

### 3.2 The only comparison you need to remember

| Question | OSI | TCP/IP |
|----------|-----|--------|
| How many layers? | 7 | 4 |
| Who uses it? | Books, interviews, discussions | The actual internet |
| Is it a real implementation? | No, a reference | Yes |

> 💡 **Tip:** In an interview: *"OSI has 7 layers and is a reference model; TCP/IP has 4 layers and is what we actually run. OSI's session, presentation and application layers are merged into the TCP/IP application layer."*

---

## 4. Encapsulation: How Data Gets Wrapped

**What it is:** Each layer adds its own header to your data on the way down, and removes it on the way up. Like putting a letter inside an envelope, inside a parcel, inside a truck.

![Encapsulation](images/03-encapsulation.png)

### 4.1 The names of the thing at each layer

| Layer | Name of the unit | What got added |
|-------|------------------|----------------|
| 7 Application | **Data** | Your actual content (`GET /index.html`) |
| 4 Transport | **Segment** (TCP) / **Datagram** (UDP) | Source port + destination port |
| 3 Network | **Packet** | Source IP + destination IP |
| 2 Data Link | **Frame** | Source MAC + destination MAC + error check |
| 1 Physical | **Bits** | Nothing - it is just signals now |

### 4.2 Why you should care

- A **packet capture** (`tcpdump`) shows you these exact headers. Once you know the layers, the output stops looking like noise.
- **MTU** (usually 1500 bytes) is the biggest frame a link can carry. Add a VPN or a tunnel and every packet carries extra headers, so the usable space shrinks. That is why VPNs sometimes break big requests but ping works fine.

```bash
# See the headers for yourself (Ctrl+C to stop)
sudo tcpdump -i any -n port 443
```

> ⚠️ **Warning:** "Ping works but the website hangs halfway" is the classic MTU problem. Ping is tiny; your HTTPS response is not.

---

## 5. What Happens When You Open a Website

This single picture ties the whole guide together. Everything you will read later is one of these steps.

![What happens when you browse](images/04-what-happens-when-you-browse.png)

### 5.1 The six steps

1. **DNS** - your computer asks "what is the IP of `example.com`?" and gets `93.184.216.34`. (Section 15)
2. **TCP handshake** - your computer and the server agree to talk, on port 443. (Section 13)
3. **TLS handshake** - they swap a certificate and keys, so everything after this is encrypted. (Section 17)
4. **HTTP request** - `GET / HTTP/1.1` plus headers goes up the tunnel. (Section 16)
5. **HTTP response** - `200 OK` comes back with HTML, then CSS, JS and images.
6. **Render** - the browser draws the page.

### 5.2 Watch it happen

```bash
# 1. DNS only
dig +short example.com
# 93.184.216.34

# 2-5. Everything, with the handshake and headers shown
curl -v https://example.com

# How long each stage took
curl -w "dns:%{time_namelookup}s connect:%{time_connect}s tls:%{time_appconnect}s total:%{time_total}s\n" \
     -o /dev/null -s https://example.com
```

Example output:

```
dns:0.021s connect:0.145s tls:0.302s total:0.418s
```

Now you can answer "why is the site slow?" with a number instead of a guess: slow DNS, slow connect (network/distance), slow TLS, or slow server.

> 💡 **Tip:** `curl -w` is the single most useful performance command in this guide. Save it as an alias.

---

## 6. IP Addressing

**What it is:** An IP address is the "postal address" of a machine on a network.

![IP address anatomy](images/05-ip-address.png)

### 6.1 IPv4 in one line

An IPv4 address is **4 numbers (octets), each 0-255**, which is really **32 bits**:

```
192.168.10.25
 │   │   │  └── octet 4 → 00011001
 │   │   └───── octet 3 → 00001010
 │   └───────── octet 2 → 10101000
 └───────────── octet 1 → 11000000
```

### 6.2 Network part and host part

Every address is split in two:

- The **network part** says *which network* (like the street name)
- The **host part** says *which machine on it* (like the house number)

Where the split happens is decided by the **subnet mask** (next section). Two machines can talk directly only if their network part is identical.

### 6.3 Three addresses you cannot give to a machine

| Address in `192.168.10.0/24` | Meaning |
|------------------------------|---------|
| `192.168.10.0` | The network itself |
| `192.168.10.255` | Broadcast - "everyone on this network" |
| `192.168.10.1` | Not reserved, but it is almost always the **gateway** (router) |

So a `/24` has 256 addresses but only **254 usable**.

### 6.4 See your own

```bash
ip a                      # Linux: all addresses
ip route                  # Linux: your gateway ("default via ...")
ipconfig                  # Windows
ifconfig                  # Mac / older Linux
hostname -I               # Linux: just the IPs
curl ifconfig.me          # your PUBLIC IP (see section 9)
```

Typical Linux output:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.1.25/24 brd 192.168.1.255 scope global dynamic eth0
```

Read it as: *my IP is `192.168.1.25`, my network is `192.168.1.0/24`, broadcast is `192.168.1.255`, and the address was given by DHCP (`dynamic`).*

### 6.5 What about IPv6?

IPv4 has about 4.3 billion addresses - not enough for the world, which is why NAT exists. IPv6 uses **128 bits** and looks like this:

```
2606:2800:220:1:248:1893:25c8:1946
fe80::1                             # :: means "a run of zeros"
::1                                 # loopback, the IPv6 version of 127.0.0.1
```

You rarely subnet IPv6 by hand, but you will see `AAAA` DNS records and `fe80::` link-local addresses on every Linux box.

> 💡 **Tip:** `127.0.0.1` (and the name `localhost`) always means *this machine*. Nothing leaves the computer. If your app only listens there, nobody else can reach it - that is one of the most common beginner bugs.

---

## 7. Subnet Mask and CIDR

**What it is:** The mask is the line that separates the network part from the host part.

![Subnet mask](images/06-subnet-mask.png)

### 7.1 Two ways to write the same thing

| CIDR | Subnet mask | Network bits | Host bits |
|------|-------------|--------------|-----------|
| `/24` | `255.255.255.0` | 24 | 8 |
| `/16` | `255.255.0.0` | 16 | 16 |
| `/8` | `255.0.0.0` | 8 | 24 |

`192.168.10.25/24` and `192.168.10.25 255.255.255.0` are **identical**. CIDR (`/24`) is just shorter, and it is what every cloud console asks for.

### 7.2 The only formula you need

```
usable hosts = 2 ^ (32 - prefix) - 2
```

The `- 2` removes the network address and the broadcast address.

| CIDR | Mask | Total addresses | Usable hosts |
|------|------|-----------------|--------------|
| `/30` | 255.255.255.252 | 4 | **2** |
| `/29` | 255.255.255.248 | 8 | **6** |
| `/28` | 255.255.255.240 | 16 | **14** |
| `/27` | 255.255.255.224 | 32 | **30** |
| `/26` | 255.255.255.192 | 64 | **62** |
| `/25` | 255.255.255.128 | 128 | **126** |
| `/24` | 255.255.255.0 | 256 | **254** |
| `/23` | 255.255.254.0 | 512 | **510** |
| `/22` | 255.255.252.0 | 1,024 | **1,022** |
| `/16` | 255.255.0.0 | 65,536 | **65,534** |
| `/8` | 255.0.0.0 | 16,777,216 | **16,777,214** |

**Smaller number = bigger network.** `/16` is huge, `/30` is tiny.

### 7.3 Two shortcuts worth memorising

- `/32` = exactly **one** address. You will use it constantly in firewall rules: `203.0.113.5/32` means "only this one IP".
- `0.0.0.0/0` = **every** address on the internet. "Allow from `0.0.0.0/0`" means "allow from anybody". Be careful with it.

### 7.4 Check it with a command

```bash
# Install if needed: sudo apt install ipcalc
ipcalc 192.168.10.25/26
```

```
Address:   192.168.10.25
Netmask:   255.255.255.192 = 26
Network:   192.168.10.0/26
HostMin:   192.168.10.1
HostMax:   192.168.10.62
Broadcast: 192.168.10.63
Hosts/Net: 62
```

> 💡 **Tip:** Nobody memorises masks. Memorise the **formula** and the three common ones (`/24`, `/16`, `/32`), then use `ipcalc`.

---

## 8. Subnetting Step by Step

**What it is:** Cutting one network into several smaller ones.

**Why DevOps people care:** Every cloud VPC is subnetted. Public subnets hold the load balancer, private subnets hold the app and database. Get it wrong and you either cannot reach your servers or you expose your database to the internet.

![Subnetting](images/07-subnetting.png)

### 8.1 The method (4 steps)

Goal: split `192.168.10.0/24` into **4 subnets**.

**Step 1 - how many bits to borrow?**
`2^n >= 4` → `n = 2`. Borrow 2 bits from the host part.

**Step 2 - new prefix**
`/24 + 2 = /26` → mask `255.255.255.192`

**Step 3 - block size**
`256 - 192 = 64`. Each subnet holds 64 addresses, so networks start every 64: `0, 64, 128, 192`.

**Step 4 - write them out**

| # | Network | Usable range | Broadcast | Hosts |
|---|---------|--------------|-----------|-------|
| 1 | `192.168.10.0/26` | .1 - .62 | .63 | 62 |
| 2 | `192.168.10.64/26` | .65 - .126 | .127 | 62 |
| 3 | `192.168.10.128/26` | .129 - .190 | .191 | 62 |
| 4 | `192.168.10.192/26` | .193 - .254 | .255 | 62 |

### 8.2 A real cloud example

A VPC of `10.0.0.0/16` (65,536 addresses) across 2 availability zones:

| Subnet | CIDR | What lives there |
|--------|------|------------------|
| Public  AZ-a | `10.0.1.0/24` | Load balancer, NAT gateway, bastion |
| Public  AZ-b | `10.0.2.0/24` | Load balancer |
| Private AZ-a | `10.0.11.0/24` | App servers / pods |
| Private AZ-b | `10.0.12.0/24` | App servers / pods |
| Data    AZ-a | `10.0.21.0/24` | Database |
| Data    AZ-b | `10.0.22.0/24` | Database replica |

> ⚠️ **Warning:** Cloud providers steal some addresses from every subnet. AWS reserves **5** per subnet (network, VPC router, DNS, future use, broadcast), so a `/24` gives you **251** usable addresses, not 254.

> 💡 **Tip:** Leave gaps (`.1`, `.2`, then `.11`, `.12`) so you can add subnets later without renumbering. Never pick `10.0.0.0/16` for one small project if the company might later connect networks together - overlapping CIDRs are painful to fix.

### 8.3 Practice (answers below)

1. How many usable hosts in `10.0.5.0/27`?
2. Which subnet does `172.16.4.130/25` belong to?
3. Split `192.168.1.0/24` into 8 subnets - what is the new prefix?

<details>
<summary>Show answers</summary>

1. `2^(32-27) - 2 = 30` usable hosts.
2. `/25` block size is 128, so networks are `.0` and `.128`. `130` falls in **`172.16.4.128/25`** (range .129 - .254, broadcast .255).
3. `2^3 = 8`, so borrow 3 bits: `/24 + 3 =` **`/27`** (block size 32: .0, .32, .64, .96, .128, .160, .192, .224).

</details>

---

## 9. Public vs Private IP

**What it is:** Private IPs are free, reusable addresses for inside your own network. Public IPs are globally unique and reachable from the internet.

![Public vs private IP](images/08-public-vs-private-ip.png)

### 9.1 The private ranges (RFC 1918) - memorise these

| Range | CIDR | Size | Where you see it |
|-------|------|------|------------------|
| `10.0.0.0` - `10.255.255.255` | `10.0.0.0/8` | 16.7M | Cloud VPCs, big companies |
| `172.16.0.0` - `172.31.255.255` | `172.16.0.0/12` | 1M | Docker default (`172.17.0.0/16`) |
| `192.168.0.0` - `192.168.255.255` | `192.168.0.0/16` | 65K | Home and office routers |

### 9.2 Special addresses

| Address | Meaning |
|---------|---------|
| `127.0.0.1` | Loopback = this machine (`localhost`) |
| `0.0.0.0` | "All interfaces" when a server binds; "any address" in a route |
| `169.254.x.x` | Link-local / APIPA - **DHCP failed**, no real IP was given |
| `169.254.169.254` | Cloud metadata service (AWS/Azure/GCP instance info) |
| `255.255.255.255` | Broadcast to everyone on this network |

### 9.3 Why this matters every day

```bash
ip a | grep inet          # 192.168.1.25  → private, inside your network
curl ifconfig.me          # 203.0.113.7   → public, what the world sees
```

If both commands show the **same** address, your machine has a real public IP (common on cloud servers). If they differ, you are behind **NAT** (next section).

> ⚠️ **Warning:** A private IP is not security. Anything inside the same network can still reach it. Firewalls and security groups are what protect you (Section 23).

> 💡 **Tip:** Interview question: *"Can two companies both use 192.168.1.10?"* Yes - private addresses are never routed on the internet, so they can be reused everywhere. The only problem is when you connect two such networks together (VPN) and the ranges overlap.

---

## 10. NAT: Network Address Translation

**What it is:** The router rewrites private source addresses into one public address, and keeps a table so replies find their way home.

**Why DevOps people care:** NAT is why your 20 home devices share one IP, and why private cloud servers need a **NAT gateway** to install packages while staying unreachable from outside.

![NAT](images/09-nat.png)

### 10.1 What the router does

```
Laptop sends:   192.168.1.10:54321 → 93.184.216.34:443
Router rewrites: 203.0.113.7:40001 → 93.184.216.34:443   and remembers the mapping
Reply comes to:  93.184.216.34:443 → 203.0.113.7:40001
Router rewrites: 93.184.216.34:443 → 192.168.1.10:54321  ✅ back to your laptop
```

### 10.2 The two types you will meet

| Type | Direction | Real example |
|------|-----------|--------------|
| **SNAT / PAT / Masquerade** | Inside → out | Home router; AWS NAT gateway letting private subnets run `apt update` |
| **DNAT / Port forwarding** | Outside → in | Public `:8080` forwarded to `192.168.1.10:80`; a Kubernetes `NodePort` |

### 10.3 In the cloud

| Setup | Can reach the internet? | Reachable from the internet? |
|-------|------------------------|------------------------------|
| Public subnet + public IP | ✅ yes | ✅ yes (if the firewall allows) |
| Private subnet + NAT gateway | ✅ yes (outbound only) | ❌ no |
| Private subnet, no NAT | ❌ no | ❌ no |

That middle row is the normal, safe place for application and database servers.

> ⚠️ **Warning:** "My server in the private subnet cannot `apt update`" is nearly always a missing NAT gateway or a missing route to it. Check the route table before blaming DNS.

> 💡 **Tip:** NAT breaks the idea that every host is directly reachable. That is why VoIP, peer-to-peer and some game protocols need extra tricks (STUN/TURN) - and why incoming connections need explicit port forwarding.

---

## 11. DHCP: Getting an IP Automatically

**What it is:** A server that hands out IP addresses and network settings to any device that joins.

![DHCP DORA](images/10-dhcp.png)

### 11.1 DORA - the 4 steps

| Step | Who talks | Message |
|------|-----------|---------|
| **D**iscover | Client (broadcast) | "Is there a DHCP server out there?" |
| **O**ffer | Server | "Yes - take `192.168.1.50`" |
| **R**equest | Client | "I accept that one" |
| **A**cknowledge | Server | "Confirmed, the lease lasts 24 hours" |

It runs over **UDP ports 67 (server) and 68 (client)** - UDP, because the client has no IP yet and cannot do a TCP handshake.

### 11.2 What you receive

```
IP address   192.168.1.50/24
Gateway      192.168.1.1
DNS servers  8.8.8.8, 1.1.1.1
Lease time   86400 seconds
```

### 11.3 Commands

```bash
# Linux
sudo dhclient -r && sudo dhclient     # release and renew
cat /var/lib/dhcp/dhclient.leases     # see the current lease

# Windows
ipconfig /release
ipconfig /renew
```

### 11.4 Static vs dynamic

| | DHCP (dynamic) | Static |
|---|---|---|
| Laptops, phones, containers | ✅ | |
| Servers, routers, printers, databases | | ✅ |
| Why | Zero admin work | The address must never change |

In the cloud you get the best of both: the instance uses DHCP, but the subnet always gives it the **same** private IP for its lifetime.

> ⚠️ **Warning:** An address starting `169.254.` means DHCP got **no answer**. Check that the DHCP server is up, the interface is really connected, and that nothing is blocking UDP 67/68.

---

## 12. Routing and Gateways

**What it is:** Routing is how a packet finds its way from one network to another. A **router** is a machine that connects networks and forwards packets between them. The **gateway** (default gateway) is the router your machine sends everything to when the destination is *not* on its own network.

**Why DevOps people care:** "Can't reach it" is very often "no route to it". Every cloud subnet has a route table, every Docker host is a router, and every Kubernetes node forwards pod traffic. If you can read a route table, you can explain most connectivity problems.

```mermaid
flowchart LR
    A["Laptop<br/>192.168.1.25/24"] -->|"destination not in 192.168.1.0/24<br/>→ send to gateway"| R["Home router / gateway<br/>192.168.1.1"]
    A -.->|"same subnet<br/>→ deliver directly (ARP)"| P["Printer<br/>192.168.1.40"]
    R -->|"NAT + forward"| ISP["ISP router"]
    ISP --> NET(("Internet"))
    NET --> S["Server<br/>93.184.216.34"]
```

### 12.1 The one decision every host makes

For every packet it sends, a machine asks one question:

```
Is the destination IP inside my own subnet?
 ├── YES → deliver it directly. Use ARP to find the destination's MAC address.
 └── NO  → send it to the gateway. Use ARP to find the GATEWAY's MAC address.
```

That is the whole idea. The gateway then makes the same kind of decision using its own route table, and so on, hop by hop, until the packet arrives. Each hop decrements the **TTL** by one; at 0 the packet is dropped (this is exactly how `traceroute` works).

### 12.2 Reading a route table

```bash
ip route                 # Linux
route print              # Windows
netstat -rn              # Mac / older Linux
```

```
default via 192.168.1.1 dev eth0 proto dhcp metric 100
10.8.0.0/24 dev wg0 proto kernel scope link src 10.8.0.6
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.25
```

| Line | Meaning |
|------|---------|
| `default via 192.168.1.1` | Anything I don't have a better route for → send to the router. `default` = `0.0.0.0/0` |
| `10.8.0.0/24 dev wg0` | The VPN network is reachable through the WireGuard interface |
| `172.17.0.0/16 dev docker0` | Docker containers live behind the `docker0` bridge |
| `192.168.1.0/24 dev eth0 scope link` | My own LAN - deliver directly, no gateway needed |

### 12.3 Longest prefix match - how the route is chosen

When several routes match, the **most specific one wins** (the biggest prefix number).

| Destination | Matching routes | Winner |
|-------------|-----------------|--------|
| `10.8.0.20` | `10.8.0.0/24`, `0.0.0.0/0` | `/24` → via `wg0` (VPN) |
| `8.8.8.8` | `0.0.0.0/0` only | default → via the router |
| `192.168.1.40` | `192.168.1.0/24`, `0.0.0.0/0` | `/24` → direct on `eth0` |

If two routes have the same prefix, the lower **metric** wins. This is why a VPN can take over "all traffic": it simply installs a more specific route (or two `/1` routes that beat `/0`).

```bash
# Ask the kernel which route it would use - the most useful routing command
ip route get 8.8.8.8
# 8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.25
```

### 12.4 Static routes

```bash
# Reach the 10.20.0.0/16 network through a router at 192.168.1.254
sudo ip route add 10.20.0.0/16 via 192.168.1.254
sudo ip route del 10.20.0.0/16

# Change the default gateway
sudo ip route replace default via 192.168.1.1

# Windows
route add 10.20.0.0 mask 255.255.0.0 192.168.1.254
```

> ⚠️ **Warning:** `ip route add` is lost on reboot. Make it permanent in netplan (`/etc/netplan/*.yaml` → `routes:`), NetworkManager (`nmcli`), or `/etc/sysconfig/network-scripts/route-eth0` on RHEL.

### 12.5 Turning a Linux box into a router

A Linux machine only forwards packets between interfaces if you allow it. Docker, Kubernetes and VPN servers all switch this on:

```bash
cat /proc/sys/net/ipv4/ip_forward               # 0 = off, 1 = on
sudo sysctl -w net.ipv4.ip_forward=1            # on until reboot
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-forward.conf   # permanent
```

Add a NAT rule (Section 24) and you have built a home router / NAT gateway.

### 12.6 Routing in the cloud

A cloud **route table** is the same idea, attached to a subnet:

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | `local` | Anything inside the VPC is delivered directly |
| `0.0.0.0/0` | `igw-xxxx` (internet gateway) | This makes it a **public** subnet |
| `0.0.0.0/0` | `nat-xxxx` (NAT gateway) | Private subnet with outbound internet |
| `172.16.0.0/16` | `pcx-xxxx` / `tgw-xxxx` | Another VPC via peering / transit gateway |
| `192.168.0.0/16` | `vgw-xxxx` | The office, via a site-to-site VPN |

More in Section 26 (VPC).

### 12.7 Static vs dynamic routing

| | Static | Dynamic |
|---|---|---|
| Who writes routes | You, by hand | Routers tell each other (protocols) |
| Protocols | - | **BGP** (between companies / the internet, cloud VPN, Kubernetes Calico), **OSPF** (inside a company) |
| Good for | Small networks, servers, cloud route tables | Large or changing networks |

> 💡 **Tip:** You will rarely configure BGP yourself, but you will hear it: "a BGP leak took down half the internet", "Calico peers with the top-of-rack switch over BGP", "the Direct Connect announces our prefixes over BGP". It is the protocol that tells the internet *where* each IP range lives.

> ⚠️ **Warning:** **Asymmetric routing** - the request goes out one path and the reply comes back another - makes stateful firewalls drop the reply. If traffic works one way but "hangs" the other, check the route back, not just the route there.

---

# PART 2: Protocols (The Languages)

---

## 13. TCP vs UDP

**What it is:** Two ways to deliver data. TCP checks everything arrived; UDP just fires it off.

![TCP vs UDP](images/11-tcp-vs-udp.png)

### 13.1 Side by side

| | TCP | UDP |
|---|-----|-----|
| Connection first? | Yes - 3-way handshake | No |
| Lost packet | Re-sent automatically | Gone forever |
| Order | Guaranteed | Not guaranteed |
| Speed | Slower (more checks) | Faster |
| Overhead per packet | 20+ bytes | 8 bytes |
| Analogy | A phone call | Shouting across a room |
| Used by | HTTP(S), SSH, SMTP, FTP, MySQL | DNS, DHCP, NTP, video calls, games, syslog |

### 13.2 The TCP 3-way handshake

```
Client ──────── SYN ────────▶ Server     "can we talk?"
Client ◀────── SYN-ACK ────── Server     "yes, can we talk?"
Client ──────── ACK ────────▶ Server     "yes" → connection established
```

And to close it: `FIN` → `ACK` → `FIN` → `ACK`.

Watch a real one:

```bash
sudo tcpdump -i any -n "tcp port 443 and host example.com"
curl -s https://example.com > /dev/null     # in another terminal
```

### 13.3 Why this explains your error messages

| What you see | What it means at TCP level |
|--------------|---------------------------|
| **Connection refused** | The SYN arrived, but nothing is listening on that port → the service is down or on another port |
| **Connection timed out** | The SYN got no answer at all → a firewall is **dropping** packets, or the host is unreachable |
| **Connection reset by peer** | The other side sent `RST` and killed the connection mid-flight |

That difference between *refused* and *timed out* saves hours. Refused = service problem. Timeout = firewall/routing problem.

### 13.4 When to pick UDP

Pick UDP when a late packet is worthless anyway: live video, voice, metrics, logs, game state. Pick TCP whenever losing a byte would corrupt the result: web pages, files, databases.

> 💡 **Tip:** HTTP/3 runs on **QUIC**, which is built on UDP - but it re-implements reliability itself. So "web = TCP" is true for HTTP/1.1 and HTTP/2, and no longer strictly true for HTTP/3.

---

## 14. Ports and Sockets

**What it is:** The IP finds the **machine**; the port finds the **program** on that machine.

![Ports](images/12-ports.png)

### 14.1 The three ranges

| Range | Name | Notes |
|-------|------|-------|
| 0 - 1023 | Well-known | Need root/admin to bind (22, 80, 443…) |
| 1024 - 49151 | Registered | Apps and databases (3306, 5432, 8080…) |
| 49152 - 65535 | Ephemeral | The random **source** port your client uses |

A **socket** is the pair `IP:port`. A connection is really four things: `source IP:port → destination IP:port`.

### 14.2 The ports every DevOps engineer should know by heart

| Port | Protocol | Use |
|------|----------|-----|
| 20 / 21 | FTP | Old file transfer (plain text) |
| 22 | SSH / SFTP / SCP | Remote login and secure copy |
| 25 | SMTP | Server-to-server mail (usually blocked in cloud) |
| 53 | DNS | Name lookups (UDP, TCP for large answers) |
| 67 / 68 | DHCP | Automatic IP configuration |
| 80 | HTTP | Web, not encrypted |
| 110 / 995 | POP3 / POP3S | Download mail |
| 143 / 993 | IMAP / IMAPS | Read mail on the server |
| 389 / 636 | LDAP / LDAPS | Directory and logins |
| 443 | HTTPS | Web over TLS |
| 587 / 465 | SMTP submission | Apps sending mail |
| 3306 | MySQL | Database |
| 3389 | RDP | Windows remote desktop |
| 5432 | PostgreSQL | Database |
| 6379 | Redis | Cache |
| 8080 / 3000 / 5000 | HTTP alternatives | Dev servers, apps behind a proxy |
| 9090 / 9100 | Prometheus / node exporter | Metrics |
| 27017 | MongoDB | Database |

### 14.3 Seeing what is listening

```bash
ss -tulpn                 # modern Linux: TCP, UDP, listening, process, numeric
netstat -tulpn            # older systems, same idea
sudo lsof -i :8080        # which process owns port 8080
```

```
Netid  State   Local Address:Port   Process
tcp    LISTEN  0.0.0.0:443          users:(("nginx",pid=812))
tcp    LISTEN  127.0.0.1:3306       users:(("mysqld",pid=640))
```

Read it carefully:

- `0.0.0.0:443` = listening on **all** interfaces → reachable from outside ✅
- `127.0.0.1:3306` = listening on **localhost only** → nothing outside this machine can connect ❌

### 14.4 Testing a port from another machine

```bash
nc -zv example.com 443           # best: tells you open / refused / timeout
telnet example.com 443           # old but everywhere
curl -v telnet://example.com:443 # when nc and telnet are missing
```

> ⚠️ **Warning:** "It works locally but not from the load balancer" is usually a `127.0.0.1` bind. Change the app to listen on `0.0.0.0` (or the specific private IP) and check again with `ss -tulpn`.

> 💡 **Tip:** Ports below 1024 need root. That is why containers and apps usually listen on 8080 and let nginx or the load balancer own 80/443.

---

## 15. DNS

**What it is:** The system that turns names into IP addresses.

**Why DevOps people care:** More outages are caused by DNS than anyone admits - wrong record, cached TTL, wrong resolver, expired domain.

![DNS](images/13-dns.png)

### 15.1 How a lookup works

1. Browser cache → 2. OS cache → 3. `/etc/hosts` → 4. **Resolver** (your ISP, `8.8.8.8`, `1.1.1.1`)
5. Resolver asks a **root** server → "ask `.com`"
6. Asks the **TLD** server → "ask `ns1.example.com`"
7. Asks the **authoritative** server → "`93.184.216.34`"
8. Answer is cached at every step for **TTL** seconds.

### 15.2 Record types

| Record | Does what | Example |
|--------|-----------|---------|
| `A` | Name → IPv4 | `example.com → 93.184.216.34` |
| `AAAA` | Name → IPv6 | `example.com → 2606:2800::1` |
| `CNAME` | Alias to another name | `www → example.com` |
| `MX` | Where email goes | `10 mail.example.com` |
| `TXT` | Free text: SPF, DKIM, domain verification | `"v=spf1 include:..."` |
| `NS` | Which servers are authoritative | `ns1.example.com` |
| `PTR` | IP → name (reverse) | `34.216.184.93.in-addr.arpa` |
| `SRV` | Service location (port + host) | Used by Kubernetes, SIP, AD |

### 15.3 The commands

```bash
dig example.com                    # full answer with TTL and sections
dig +short example.com             # just the IP - use this 90% of the time
dig MX example.com                 # a specific record type
dig @8.8.8.8 example.com           # ask one specific resolver
dig +trace example.com             # follow root → TLD → authoritative yourself
dig -x 93.184.216.34               # reverse lookup

nslookup example.com               # works on Windows too
host example.com                   # short and simple

cat /etc/resolv.conf               # which resolver your Linux box uses
resolvectl status                  # systemd machines
ipconfig /displaydns               # Windows: see the cache
ipconfig /flushdns                 # Windows: clear it
```

### 15.4 Reading `dig` output

```
;; ANSWER SECTION:
example.com.        3600    IN    A     93.184.216.34
                     │             │      └── the answer
                     │             └── record type
                     └── TTL: cached for 3600 seconds (1 hour)
```

### 15.5 The classic DNS traps

| Symptom | Cause | Fix |
|---------|-------|-----|
| Old IP still answering after a migration | TTL caching | Lower the TTL to 60s **before** the change, raise it after |
| Works on one machine, not another | Different resolvers, or a stale `/etc/hosts` entry | `dig @8.8.8.8` vs `dig` and compare |
| `curl` fails but `ping` works | Different name, or IPv6 (`AAAA`) record that is broken | `dig A` and `dig AAAA` separately |
| Site dies suddenly for everyone | Domain or certificate expired | Check the registrar, then `curl -vI` |

> 💡 **Tip:** `/etc/hosts` (or `C:\Windows\System32\drivers\etc\hosts`) beats DNS. It is the fastest way to test a new server before you move the real record:
> ```
> 203.0.113.50   app.example.com
> ```

> ⚠️ **Warning:** A `CNAME` cannot sit at the root of a domain (`example.com` itself) in standard DNS. Use an `A` record, or your provider's special "ALIAS" / "ANAME" record.

---

## 16. HTTP and HTTPS

**What it is:** HTTP is the request/response language of the web. HTTPS is exactly the same language inside a TLS-encrypted tunnel on port 443.

![HTTP and HTTPS](images/14-http-https.png)

### 16.1 A request and a response

```http
GET /api/users HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOi...
User-Agent: curl/8.5.0
Accept: application/json
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 142
Cache-Control: max-age=60

{"users": [...]}
```

### 16.2 Methods

| Method | Means | Changes data? |
|--------|-------|---------------|
| `GET` | Read something | No |
| `POST` | Create something | Yes |
| `PUT` | Replace something | Yes |
| `PATCH` | Update part of something | Yes |
| `DELETE` | Remove something | Yes |
| `HEAD` | Like GET but headers only | No |

### 16.3 Status codes - what to check first

| Code | Meaning | Where to look |
|------|---------|---------------|
| `200 OK` | Fine | - |
| `301 / 302` | Moved | Redirect loop? Check your proxy rules |
| `304 Not Modified` | Use your cache | Normal |
| `400` | Bad request | The client sent nonsense |
| `401` | Not authenticated | Missing or expired token |
| `403` | Authenticated but not allowed | Permissions, WAF, S3 policy |
| `404` | Not found | Wrong path or wrong virtual host |
| `429` | Too many requests | Rate limiting |
| `500` | Server crashed | **Application logs** |
| `502 Bad Gateway` | Proxy reached but backend did not answer | **Backend is down or on the wrong port** |
| `503 Service Unavailable` | No healthy backend / overloaded | Health checks, scaling |
| `504 Gateway Timeout` | Backend was too slow | Timeouts, slow queries |

> 💡 **Tip:** `4xx` = the client's fault. `5xx` = the server's fault. `502/503/504` always point at the **gap between the proxy and the backend** - start there, not in the browser.

### 16.4 HTTPS and TLS in one minute

1. Client says hello and lists the ciphers it supports.
2. Server sends its **certificate** (proving it owns the domain).
3. They agree on a session key.
4. Everything after that is encrypted. 🔒

The full step-by-step handshake, certificates and chains are in **Section 17**.

```bash
# Look at a certificate: who issued it, when it expires
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates

# Headers only
curl -I https://example.com

# Full handshake + headers
curl -v https://example.com
```

| TLS error you will meet | Real cause |
|-------------------------|------------|
| `certificate has expired` | Nobody renewed it (automate with certbot / ACM) |
| `self signed certificate` | Internal CA not trusted by the client |
| `hostname mismatch` | Certificate is for `www.example.com`, you asked for `example.com` |
| `unable to get local issuer certificate` | Missing intermediate certificate in the chain |

### 16.5 Useful curl flags

```bash
curl -I URL                       # headers only
curl -L URL                       # follow redirects
curl -k URL                       # ignore certificate errors (debug only!)
curl -X POST -d '{"a":1}' -H "Content-Type: application/json" URL
curl -o file.zip URL              # save to a file
curl -s -o /dev/null -w "%{http_code}\n" URL    # print just the status code
curl --resolve app.example.com:443:203.0.113.50 https://app.example.com
```

That last one is gold: it tests a **new server** using the real hostname and certificate, without touching DNS.

---

## 17. SSL/TLS Certificates and the HTTPS Handshake

**What it is:** **TLS** (Transport Layer Security) is the encryption layer under HTTPS, SMTPS, IMAPS, LDAPS, database connections and more. **SSL** is its old, broken predecessor - people still say "SSL certificate", but everything today is TLS 1.2 or TLS 1.3. A **certificate** is a signed file that proves "this public key really belongs to `example.com`".

**Why DevOps people care:** Expired certificates are one of the most common *self-inflicted* outages. You will issue certificates, renew them automatically, install them on nginx/load balancers/Kubernetes ingress, and debug handshake errors.

### 17.1 TLS gives you three things

| Guarantee | Meaning | Without it |
|-----------|---------|-----------|
| **Encryption** | Nobody in the middle can read the traffic | Passwords readable on café Wi-Fi |
| **Authentication** | You are really talking to `example.com` | A fake server can impersonate the site |
| **Integrity** | Nobody changed the data in transit | Ads or malware injected into pages |

### 17.2 What is inside a certificate

```bash
echo | openssl s_client -connect github.com:443 -servername github.com 2>/dev/null \
  | openssl x509 -noout -text | less
```

| Field | Example | Why it matters |
|-------|---------|----------------|
| **Subject (CN)** | `CN=github.com` | Who the certificate is for (legacy field) |
| **Subject Alternative Names (SAN)** | `DNS:github.com, DNS:www.github.com` | **The names browsers actually check** |
| **Issuer** | `Sectigo ECC Domain Validation Secure Server CA` | Which CA signed it |
| **Validity** | `notBefore` / `notAfter` | The expiry date that wakes you up at 3am |
| **Public key** | EC 256-bit / RSA 2048-bit | Used in the handshake |
| **Signature** | Signed by the issuer's private key | Proves the CA vouches for it |

### 17.3 The chain of trust

```mermaid
flowchart TD
    R["🏛️ Root CA<br/>(pre-installed in your OS / browser trust store)"] -->|signs| I["Intermediate CA<br/>(sent by the server)"]
    I -->|signs| L["Leaf / server certificate<br/>CN=example.com<br/>(sent by the server)"]
```

1. The server sends its **leaf** certificate **plus the intermediate(s)** - this bundle is `fullchain.pem`.
2. The client checks each signature up the chain until it reaches a **root** it already trusts.
3. The client checks the name matches, the dates are valid, and it is not revoked.

> ⚠️ **Warning:** "Works in Chrome, fails in `curl` / Java / Python" is almost always a **missing intermediate**. Browsers sometimes fetch it themselves; other clients don't. Always configure `fullchain.pem`, not `cert.pem`.

### 17.4 Certificate types

| Type | Covers | Note |
|------|--------|------|
| **Single domain** | `app.example.com` | |
| **Wildcard** | `*.example.com` | One level only: covers `api.example.com`, **not** `v1.api.example.com` and not `example.com` itself |
| **SAN / multi-domain** | Several names in one cert | What Let's Encrypt gives you with multiple `-d` flags |
| **DV** (domain validated) | Proves you control the domain | Free and automatic (Let's Encrypt, AWS ACM). This is what 99% of sites use |
| **OV / EV** | Also checks the organisation | Costs money, browsers no longer show a visible difference |
| **Self-signed** | Signed by itself | Fine for labs and internal testing, untrusted everywhere else |
| **Private CA** | Your company's own root | Internal services, service mesh, mTLS |

### 17.5 File formats you will be handed

| Extension | What it is |
|-----------|-----------|
| `.pem` / `.crt` / `.cer` | Base64 text starting with `-----BEGIN CERTIFICATE-----` (most common on Linux) |
| `.key` | The **private key** (`-----BEGIN PRIVATE KEY-----`) - never share, never commit |
| `.csr` | Certificate Signing Request - what you send to a CA |
| `.der` | Binary version of a certificate (Java, Windows) |
| `.pfx` / `.p12` | Certificate + chain + private key in one password-protected file (Windows / IIS / Azure) |

```bash
# Convert PFX → PEM (cert + key)
openssl pkcs12 -in site.pfx -out site.pem -nodes

# Does this key belong to this certificate? (the two hashes must match)
openssl x509 -noout -pubkey -in cert.pem | openssl sha256
openssl pkey -pubout -in private.key   | openssl sha256
```

### 17.6 The TLS 1.3 handshake, step by step

TLS 1.3 needs **one round trip** (1-RTT) after the TCP handshake:

```mermaid
sequenceDiagram
    participant C as Client (browser)
    participant S as Server (example.com)
    Note over C,S: TCP 3-way handshake first (SYN, SYN-ACK, ACK)
    C->>S: ClientHello: TLS versions, cipher suites, key_share, SNI = example.com
    S->>C: ServerHello: chosen cipher + server key_share
    Note over C,S: Both sides now compute the same session keys (ECDHE)
    S->>C: 🔒 Certificate + CertificateVerify + Finished
    Note over C: Verify chain, name, dates
    C->>S: 🔒 Finished
    C->>S: 🔒 GET / HTTP/1.1 (application data)
    S->>C: 🔒 HTTP/1.1 200 OK
```

In plain words:

1. **ClientHello** - "I speak TLS 1.3 and 1.2, here are the ciphers I know, here is my half of a key exchange, and I want **`example.com`**" (that last bit is **SNI**).
2. **ServerHello** - "Let's use TLS 1.3 with `TLS_AES_128_GCM_SHA256`, here is my half of the key exchange."
3. Both sides combine the halves (**ECDHE** - Diffie-Hellman) into the same secret **without ever sending it**. From here on, everything is encrypted.
4. The server sends its **certificate**, and **signs** the handshake with its private key to prove it owns the certificate.
5. The client validates the certificate. Both send **Finished**. Application data flows.

**TLS 1.2 vs 1.3:**

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Round trips | 2 | **1** (0-RTT possible for repeat visits) |
| Certificate sent | In plain text | **Encrypted** |
| Old weak ciphers (RSA key exchange, CBC, SHA-1) | Allowed | **Removed** |
| Forward secrecy | Optional | **Always** |
| Status | Still acceptable | Preferred |
| SSL 3.0, TLS 1.0, TLS 1.1 | ❌ Disable everywhere | |

> 💡 **Tip:** **Forward secrecy** means that even if someone steals the server's private key next year, they cannot decrypt traffic they recorded today - because the session keys came from a temporary Diffie-Hellman exchange, not from the certificate key.

### 17.7 SNI - many HTTPS sites on one IP

The client sends the hostname in the ClientHello **before** encryption starts, so one nginx or load balancer on one IP can pick the right certificate for hundreds of domains. This is why `openssl s_client` needs `-servername`, and why a missing SNI gives you the *default* (wrong) certificate.

```bash
# With SNI - correct certificate
openssl s_client -connect 203.0.113.50:443 -servername app.example.com
# Without SNI - you may see the default certificate instead
openssl s_client -connect 203.0.113.50:443
```

### 17.8 Get a free certificate with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com   # issues AND edits nginx for you
sudo certbot certificates                                # what you have, and expiry dates
sudo certbot renew --dry-run                             # test auto-renewal
systemctl list-timers | grep certbot                     # the renewal timer
```

Let's Encrypt certificates last **90 days**; certbot renews them automatically around day 60. Other ways to get them: **AWS ACM** (free, auto-renewing, for ALB/CloudFront), **cert-manager** in Kubernetes, Caddy/Traefik (built-in ACME).

How Let's Encrypt checks you own the domain:

| Challenge | How | Use when |
|-----------|-----|----------|
| **HTTP-01** | Serves a file at `http://domain/.well-known/acme-challenge/` | Normal public web server (port 80 must be open) |
| **DNS-01** | Creates a `TXT` record `_acme-challenge.domain` | **Wildcard** certificates, servers not reachable from the internet |

### 17.9 Make your own (labs and internal use)

```bash
# Self-signed certificate + key in one command (valid 365 days, with a SAN)
openssl req -x509 -newkey rsa:2048 -nodes -days 365 \
  -keyout server.key -out server.crt \
  -subj "/CN=myapp.local" -addext "subjectAltName=DNS:myapp.local,IP:127.0.0.1"

# A CSR to send to a real CA
openssl req -new -newkey rsa:2048 -nodes \
  -keyout example.com.key -out example.com.csr -subj "/CN=example.com"
```

### 17.10 nginx TLS config that is actually good

```nginx
server {
    listen 443 ssl;
    http2 on;                  # nginx 1.25.1+; older versions: "listen 443 ssl http2;"
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;   # leaf + intermediates
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;   # HSTS
}

server {                       # redirect all plain HTTP to HTTPS
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

**HSTS** tells the browser "never talk plain HTTP to this site again", which blocks downgrade attacks. Grade the result with the [SSL Labs test](https://www.ssllabs.com/ssltest/).

### 17.11 mTLS - when the client needs a certificate too

Normal TLS: only the **server** proves who it is. **Mutual TLS**: the **client** also presents a certificate. Used for service-to-service traffic (Istio/Linkerd service mesh), APIs between companies, and zero-trust networks.

```bash
curl --cert client.crt --key client.key --cacert ca.crt https://api.internal
```

### 17.12 Where TLS terminates

| Pattern | Traffic is encrypted | Note |
|---------|---------------------|------|
| **Termination at the LB** | Client → LB only | Simplest; backends speak plain HTTP inside the VPC |
| **Re-encryption** | Client → LB, and LB → backend | Required by many compliance rules (PCI, HIPAA) |
| **Passthrough** (L4) | Client → backend directly | LB cannot read HTTP, so no path routing; backend holds the cert |

### 17.13 Monitoring expiry

```bash
# Days left on a certificate - put this in a cron job or monitoring check
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -enddate

# Fail if it expires in the next 14 days (exit code 1)
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -checkend $((14*24*3600))
```

The common TLS error messages and their real causes are in the table in **16.4**.

> ⚠️ **Warning:** `curl -k` / `verify=False` / `InsecureSkipVerify: true` turns off authentication completely - you are encrypted, but possibly to an attacker. Use it to debug for one minute, never in code or pipelines. Fix the trust store (`--cacert`) instead.

---

## 18. SSH

**What it is:** Encrypted remote login and file copy, on **port 22**.

![SSH](images/15-ssh.png)

### 18.1 Keys, not passwords

```bash
ssh-keygen -t ed25519 -C "you@example.com"     # creates ~/.ssh/id_ed25519(.pub)
ssh-copy-id user@10.0.1.5                      # copies the PUBLIC key to the server
ssh user@10.0.1.5                              # log in, no password
```

- `id_ed25519` = **private** key. Never leaves your laptop, never goes in Git.
- `id_ed25519.pub` = **public** key. Goes into the server's `~/.ssh/authorized_keys`.

### 18.2 Everyday commands

```bash
ssh -i ~/.ssh/aws.pem ec2-user@203.0.113.9     # specific key file
ssh -p 2222 user@host                          # non-default port
ssh user@host "df -h"                          # run one command and exit
scp report.txt user@host:/tmp/                 # copy a file up
scp user@host:/var/log/app.log .               # copy a file down
rsync -avz ./site/ user@host:/var/www/site/    # copy only what changed
sftp user@host                                 # interactive file session
```

### 18.3 SSH tunnels (how you reach a locked-down service)

```bash
# Local forward: my localhost:8080 → the server's localhost:80
ssh -L 8080:localhost:80 user@bastion

# Reach a private database through a bastion host
ssh -L 5432:db.internal:5432 user@bastion
# then: psql -h localhost -p 5432
```

This is how you open a private admin panel or database that has no public IP at all.

### 18.4 `~/.ssh/config` - stop typing long commands

```
Host prod
    HostName 203.0.113.9
    User ec2-user
    IdentityFile ~/.ssh/aws.pem
    Port 22

Host db
    HostName 10.0.21.10
    User admin
    ProxyJump prod          # hop through the bastion automatically
```

Now `ssh prod` and `ssh db` just work.

### 18.5 Hardening (`/etc/ssh/sshd_config`)

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

```bash
sudo systemctl restart sshd
```

Plus: allow port 22 **only** from your office or VPN range in the firewall.

> ⚠️ **Warning:** `chmod 600 ~/.ssh/id_ed25519` - SSH refuses keys that other users can read. And if you ever commit a private key or a `.pem` to Git, treat it as compromised and rotate it.

> 💡 **Tip:** `ssh -v user@host` shows exactly where the login fails: TCP connect, key offered, or permission. Read the first failure, not the last line.

---

## 19. FTP, SFTP and File Transfer

![File transfer and email protocols](images/16-ftp-smtp.png)

| Protocol | Port | Encrypted? | Use it? |
|----------|------|-----------|---------|
| **FTP** | 21 (+20 data) | ❌ No - password in plain text | ❌ Avoid |
| **FTPS** | 990 / 21 + TLS | ✅ Yes | Only for legacy systems |
| **SFTP** | 22 | ✅ Yes (over SSH) | ✅ **Default choice** |
| **SCP** | 22 | ✅ Yes (over SSH) | ✅ Quick one-off copies |
| **rsync over SSH** | 22 | ✅ Yes | ✅ Best for backups and big folders |

```bash
sftp user@host                     # interactive: put, get, ls, cd
scp -r ./dist user@host:/var/www/  # recursive copy
rsync -avz --delete ./dist/ user@host:/var/www/dist/   # mirror a folder
```

**Active vs passive FTP** (why FTP annoys firewalls): in *active* mode the **server** opens a second connection back to the client, which NAT and firewalls block. *Passive* mode lets the client open both connections, so it mostly works - but it needs a whole port range open. SFTP needs one port: 22. That is the whole argument.

> 💡 **Tip:** `rsync -avz --dry-run` first. It shows exactly what would be copied or deleted, and has saved many people from wiping a production folder.

---

## 20. SMTP and Email

| Protocol | Port | Job |
|----------|------|-----|
| **SMTP** | 25 | Server → server delivery (cloud providers usually block 25) |
| **SMTP submission** | 587 (STARTTLS) or 465 (TLS) | Your app → mail server |
| **IMAP** | 143 / 993 | Read mail, stays on the server, syncs across devices |
| **POP3** | 110 / 995 | Download mail, usually deletes the server copy |

### 20.1 Why DevOps engineers care

Alerting, password resets and CI notifications all go through SMTP. When mail stops arriving, check in this order:

1. **Egress** - can the server reach port 587? `nc -zv smtp.provider.com 587`
2. **MX record** - `dig MX example.com`
3. **SPF / DKIM / DMARC** - `dig TXT example.com` (missing records = silently sent to spam)
4. **Credentials and logs** - `/var/log/mail.log`

```bash
# Test that the SMTP port is reachable at all
nc -zv smtp.gmail.com 587

# Send a test mail from a server
echo "test body" | mail -s "test subject" you@example.com
```

> ⚠️ **Warning:** AWS, GCP and Azure block outbound port **25** by default to stop spam. Use a mail provider (SES, SendGrid, Mailgun) on port 587 or 465 instead of running your own relay.
---

# PART 3: Network Infrastructure

---

## 21. Load Balancers

**What it is:** One public entry point that spreads traffic across many servers and skips the unhealthy ones.

**Why DevOps people care:** It is what makes "deploy with no downtime" and "scale out" possible.

![Load balancer](images/17-load-balancer.png)

### 21.1 What it actually gives you

| Benefit | How |
|---------|-----|
| No single point of failure | A dead server fails its health check and gets no traffic |
| Scale out | Add servers behind the same address |
| Zero-downtime deploys | Drain one server, update it, put it back, repeat |
| TLS termination | Certificates live on the LB, backends speak plain HTTP |
| One DNS name | `app.example.com` never changes, servers come and go |

### 21.2 Layer 4 vs Layer 7

| | Layer 4 (transport) | Layer 7 (application) |
|---|---|---|
| Decides using | IP + port | URL, host header, cookies |
| Can route `/api` separately? | ❌ No | ✅ Yes |
| Speed | Very fast | Slightly slower |
| Examples | AWS NLB, HAProxy TCP mode | AWS ALB, nginx, Traefik, Envoy |

### 21.3 Algorithms

| Algorithm | How it picks |
|-----------|--------------|
| **Round robin** | Next server in the list, in turn |
| **Least connections** | The server with the fewest open connections |
| **IP hash / sticky** | The same client always lands on the same server (for in-memory sessions) |
| **Weighted** | Bigger servers get more traffic |

### 21.4 Health checks - the part people get wrong

```
GET /health  every 10s
healthy threshold:   2 successes  → put it back in
unhealthy threshold: 2 failures   → take it out
timeout: 5s
```

Your app should expose a real `/health` endpoint that checks its own dependencies and returns `200` or `503`.

```bash
# Is the LB sending me traffic? Check from the LB's point of view:
curl -s -o /dev/null -w "%{http_code}\n" http://10.0.1.11:8080/health
```

> ⚠️ **Warning:** A `503` from the load balancer with no backend logs means **no healthy targets**. Check the health check path, port and the security group between the LB and the instances - in that order.

> 💡 **Tip:** Sticky sessions are a crutch. Store sessions in Redis or a database and any server can serve any user - that is what makes autoscaling painless.

### 21.5 L4 vs L7 in real config

The same nginx can do both. The difference is *what it can see*.

**Layer 4** - nginx only sees IP + port, so it just forwards TCP bytes (works for anything: databases, MQTT, TLS passthrough):

```nginx
# /etc/nginx/nginx.conf  (top level, NOT inside http {})
stream {
    upstream postgres_pool {
        least_conn;
        server 10.0.21.10:5432;
        server 10.0.22.10:5432 backup;
    }
    server {
        listen 5432;
        proxy_pass postgres_pool;
    }
}
```

**Layer 7** - nginx reads the HTTP request, so it can route by host, path, header or cookie:

```nginx
http {
    upstream web_pool {
        server 10.0.11.10:8080 weight=3;
        server 10.0.12.10:8080;
        server 10.0.13.10:8080 max_fails=3 fail_timeout=30s;
    }
    upstream api_pool {
        server 10.0.11.20:5000;
        server 10.0.12.20:5000;
    }
    server {
        listen 80;
        location /api/ { proxy_pass http://api_pool; }   # path-based routing
        location /     { proxy_pass http://web_pool; }
    }
}
```

The same idea in **HAProxy** (very common as a dedicated LB):

```
frontend http_in
    bind *:80
    mode http                                  # L7 ("mode tcp" = L4)
    acl is_api path_beg /api
    use_backend api_servers if is_api
    default_backend web_servers

backend web_servers
    mode http
    balance roundrobin
    option httpchk GET /health
    server web1 10.0.11.10:8080 check
    server web2 10.0.12.10:8080 check

backend api_servers
    mode http
    balance leastconn
    server api1 10.0.11.20:5000 check
```

**Which to choose:**

| Pick **L4** when | Pick **L7** when |
|------------------|------------------|
| Non-HTTP traffic (databases, gRPC passthrough, game servers, MQTT) | You need path / host routing (`/api`, `api.example.com`) |
| You need the lowest latency and millions of connections | You want TLS termination, WAF, header rewriting, redirects |
| The backend must keep the client's TLS connection (passthrough) | Canary / blue-green by header or cookie |
| You need the client's real source IP without headers | You want HTTP-aware health checks and retries |

| In the cloud | L4 | L7 |
|--------------|----|----|
| AWS | Network Load Balancer (NLB) | Application Load Balancer (ALB) |
| Azure | Azure Load Balancer | Application Gateway / Front Door |
| GCP | Network Load Balancer | HTTP(S) Load Balancer |
| Kubernetes | `Service type: LoadBalancer` | `Ingress` / Gateway API |

---

## 22. Proxy and Reverse Proxy

**What it is:** A middleman. The only difference is *whose* side it sits on.

![Forward vs reverse proxy](images/18-proxy-vs-reverse-proxy.png)

| | Forward proxy | Reverse proxy |
|---|---|---|
| Sits in front of | **Clients** | **Servers** |
| Hides | The client from the server | The server from the client |
| Who configures it | The user / the company laptop | The infrastructure team |
| Typical use | Web filtering, caching downloads, controlled egress | TLS, load balancing, caching, rate limiting |
| Examples | Squid, corporate proxy, `HTTP_PROXY` env var | nginx, Traefik, HAProxy, Cloudflare, ALB |

### 22.1 Forward proxy in daily life

```bash
export HTTP_PROXY=http://proxy.company.com:3128
export HTTPS_PROXY=http://proxy.company.com:3128
export NO_PROXY=localhost,127.0.0.1,.internal
curl https://example.com        # now goes through the proxy
```

If `apt`, `docker pull` or `pip` fail behind a corporate network, those three variables (plus Docker's own proxy config) are usually the fix.

### 22.2 Reverse proxy - the config you will write a hundred times

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;

    ssl_certificate     /etc/letsencrypt/live/app.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:5000/;
    }
}
```

```bash
sudo nginx -t              # ALWAYS test before reloading
sudo systemctl reload nginx
```

Those `X-Forwarded-*` headers matter: without them your application logs every request as coming from the proxy, and redirects break by sending users to `http://` instead of `https://`.

> 💡 **Tip:** A load balancer *is* a reverse proxy whose main job is spreading traffic. nginx, Traefik and Envoy do both jobs at once - the words overlap, and that is fine.

> ⚠️ **Warning:** `502 Bad Gateway` from nginx = nginx is alive but the backend is not answering on `proxy_pass`. Check the backend is running, on the right port, and listening on an address nginx can reach.

---

## 23. Firewalls and Security Groups

**What it is:** Rules that decide which traffic is allowed in or out. The golden rule is **default deny**: block everything, then open exactly what you need.

![Firewall](images/19-firewall.png)

### 23.1 A sane rule set for a web server

| # | Direction | Port | Source | Action | Why |
|---|-----------|------|--------|--------|-----|
| 1 | Inbound | 443 | `0.0.0.0/0` | ALLOW | The public website |
| 2 | Inbound | 80 | `0.0.0.0/0` | ALLOW | Only to redirect to HTTPS |
| 3 | Inbound | 22 | `203.0.113.5/32` | ALLOW | SSH from the office/VPN only |
| 4 | Inbound | 3306 | `10.0.1.0/24` | ALLOW | MySQL from the app subnet only |
| 5 | Inbound | ANY | `0.0.0.0/0` | **DENY** | The implicit rule at the bottom |

### 23.2 Stateful vs stateless

- **Stateful** (security groups, `ufw`, `firewalld`): allow a connection in, and the reply is automatically allowed back out. You only write one rule.
- **Stateless** (AWS NACLs, plain `iptables` without conntrack): you must allow the return traffic too - including the ephemeral port range `1024-65535`.

### 23.3 Cloud wording

| Provider | Instance level | Subnet level |
|----------|----------------|--------------|
| AWS | Security group (stateful, allow-only) | NACL (stateless, allow + deny) |
| Azure | NSG | NSG on the subnet |
| GCP | Firewall rules (by tag) | VPC firewall rules |

### 23.4 Commands

```bash
# ufw (Ubuntu)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 443/tcp
sudo ufw allow from 203.0.113.5 to any port 22 proto tcp
sudo ufw enable
sudo ufw status numbered

# firewalld (RHEL / Fedora)
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all

# iptables (the low level under both)
sudo iptables -L -n -v
```

These are just the basics. **Section 24** goes deep on iptables, ufw and firewalld.

### 23.5 How a firewall feels from the outside

| Behaviour | What the firewall did |
|-----------|----------------------|
| `Connection refused` **immediately** | Packet got through; nothing is listening (or the rule REJECTs) |
| **Hangs, then timeout** | The packet was silently **DROPped** - classic firewall block |

```bash
nc -zv app.example.com 443        # succeeded / refused / timed out
```

> ⚠️ **Warning:** Never leave SSH (22) open to `0.0.0.0/0` on a production server. Bots find it within minutes. Restrict it to your IP, a bastion, or a VPN.

> 💡 **Tip:** Before you enable a firewall on a remote machine, make sure the SSH rule is in place - otherwise you lock yourself out of your own server. Open a second SSH session as a safety net while you test.

---

## 24. Linux Firewalls: iptables, ufw, firewalld

**What it is:** Linux has one firewall engine in the kernel, called **netfilter**. Everything else is a tool for writing rules into it.

```mermaid
flowchart TD
    U["ufw<br/>(Ubuntu / Debian)"] --> IPT
    F["firewalld<br/>(RHEL / Rocky / Fedora)"] --> NFT
    D["Docker / Kubernetes kube-proxy"] --> IPT
    IPT["iptables<br/>(classic rule tool)"] --> NF
    NFT["nftables / nft<br/>(modern replacement)"] --> NF
    NF["⚙️ netfilter - in the Linux kernel"]
```

| Tool | Level | Default on | Think of it as |
|------|-------|-----------|----------------|
| **iptables** | Low level | Older distros, still everywhere | The assembly language of Linux firewalls |
| **nftables** (`nft`) | Low level | Debian 10+, RHEL 8+ | The modern replacement for iptables |
| **ufw** | Friendly front-end | Ubuntu | "Uncomplicated" - a few simple commands |
| **firewalld** | Friendly front-end with **zones** | RHEL, Rocky, Alma, Fedora, CentOS | Dynamic rules without restarts |

> ⚠️ **Warning:** Use **one** front-end per machine. Running ufw and firewalld together (or hand-written iptables rules under them) creates rules that fight each other.

### 24.1 iptables - how it thinks

**Tables** group rules by purpose; **chains** are the points a packet passes through.

| Table | Job | Chains |
|-------|-----|--------|
| `filter` (default) | Allow or block | `INPUT`, `FORWARD`, `OUTPUT` |
| `nat` | Rewrite addresses (NAT, port forwarding) | `PREROUTING`, `POSTROUTING`, `OUTPUT` |
| `mangle` | Change packet fields (marks, TTL) | All five |
| `raw` | Skip connection tracking | `PREROUTING`, `OUTPUT` |

```mermaid
flowchart LR
    IN(["Packet arrives"]) --> PRE["PREROUTING<br/>(DNAT happens here)"]
    PRE --> Q{"For this machine?"}
    Q -->|yes| INPUT["INPUT"] --> APP["Local process<br/>(nginx, sshd)"]
    APP --> OUTPUT["OUTPUT"] --> POST
    Q -->|"no - forward it"| FWD["FORWARD<br/>(routers, Docker, K8s)"] --> POST["POSTROUTING<br/>(SNAT / MASQUERADE here)"]
    POST --> OUT(["Packet leaves"])
```

| Chain | Handles traffic that is… |
|-------|--------------------------|
| `INPUT` | coming **to** this machine (someone SSHing in) |
| `OUTPUT` | created **by** this machine (`apt update`) |
| `FORWARD` | passing **through** this machine (it is acting as a router / Docker host) |

**Targets** - what to do when a rule matches:

| Target | Effect | Client sees |
|--------|--------|-------------|
| `ACCEPT` | Let it through | Normal |
| `DROP` | Silently throw it away | **Timeout** |
| `REJECT` | Throw it away and tell the sender | **Connection refused** |
| `LOG` | Write to the kernel log, then continue to the next rule | - |
| `MASQUERADE` / `SNAT` / `DNAT` | Rewrite addresses (nat table) | - |

**Rules are checked top to bottom, and the first match wins.** If nothing matches, the chain's **policy** (`ACCEPT` or `DROP`) decides.

### 24.2 iptables - reading rules

```bash
sudo iptables -L -n -v --line-numbers     # filter table, numeric, with counters
sudo iptables -t nat -L -n -v             # the nat table
sudo iptables -S                          # rules printed as the commands that created them
```

```
Chain INPUT (policy DROP 120 packets, 7200 bytes)
num  pkts bytes target  prot opt in  out  source        destination
1    5012  420K ACCEPT  all  --  lo  *    0.0.0.0/0     0.0.0.0/0
2    9.1K  12M  ACCEPT  all  --  *   *    0.0.0.0/0     0.0.0.0/0    ctstate RELATED,ESTABLISHED
3      14   840 ACCEPT  tcp  --  *   *    203.0.113.5   0.0.0.0/0    tcp dpt:22
4    3311  198K ACCEPT  tcp  --  *   *    0.0.0.0/0     0.0.0.0/0    tcp dpt:443
```

The `pkts` counter tells you whether a rule is actually being hit - the fastest way to find out which rule is blocking you.

### 24.3 iptables - a complete, safe web-server rule set

```bash
#!/bin/bash
# Order matters! Allow SSH BEFORE setting the DROP policy, or you lock yourself out.

# 1. Start clean
iptables -F                      # flush all rules
iptables -X                      # delete custom chains

# 2. Always allow loopback and replies to connections we started
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# 3. SSH only from the office / VPN IP
iptables -A INPUT -p tcp -s 203.0.113.5/32 --dport 22 -j ACCEPT

# 4. Public web
iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT

# 5. MySQL only from the app subnet
iptables -A INPUT -p tcp -s 10.0.11.0/24 --dport 3306 -j ACCEPT

# 6. Allow ping (helpful for monitoring)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# 7. Log what is about to be dropped (rate-limited so logs don't explode)
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "IPT-DROP: "

# 8. Default policies LAST
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

The `ESTABLISHED,RELATED` rule is what makes iptables **stateful**: once a connection is allowed in, its replies flow automatically.

### 24.4 iptables - everyday edits

```bash
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT      # -A = append to the END
sudo iptables -I INPUT 1 -s 198.51.100.7 -j DROP          # -I = insert at position 1 (top)
sudo iptables -D INPUT 4                                  # delete rule number 4
sudo iptables -D INPUT -p tcp --dport 8080 -j ACCEPT      # delete by exact spec
sudo iptables -R INPUT 3 -p tcp -s 203.0.113.9 --dport 22 -j ACCEPT   # replace rule 3
sudo journalctl -k | grep IPT-DROP                        # see what got dropped
```

> ⚠️ **Warning:** `-A` appends **after** existing rules. If an earlier rule already drops the traffic, your new `ACCEPT` is never reached. Use `-I` to insert at the top, then check with `--line-numbers`.

### 24.5 iptables - NAT and port forwarding

```bash
# Turn this box into a NAT gateway for the 10.0.11.0/24 subnet (needs ip_forward=1, Section 12)
sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -s 10.0.11.0/24 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Port forwarding (DNAT): public :8080 → internal 10.0.11.10:80
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to-destination 10.0.11.10:80
sudo iptables -A FORWARD -p tcp -d 10.0.11.10 --dport 80 -j ACCEPT

# Redirect port 80 to an app on 8080 on the same machine
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### 24.6 iptables - make it survive a reboot

iptables rules live in memory and **vanish on reboot** unless you save them.

```bash
# Debian / Ubuntu
sudo apt install iptables-persistent
sudo netfilter-persistent save          # writes /etc/iptables/rules.v4 and rules.v6

# RHEL family (if not using firewalld)
sudo iptables-save > /etc/sysconfig/iptables

# Manual backup / restore - do this before every big change
sudo iptables-save > ~/iptables-backup.rules
sudo iptables-restore < ~/iptables-backup.rules
```

> ⚠️ **Warning:** IPv6 has its own separate rule set: **`ip6tables`**. Locking down IPv4 while IPv6 is wide open is a real and common mistake.

### 24.7 Docker and iptables (the trap everyone hits)

Docker writes its own iptables rules (in the `nat` table and the `DOCKER` chain) when you publish a port with `-p 8080:80`. Those rules are processed **before** ufw's rules, so:

> ⚠️ **Warning:** `docker run -p 3306:3306 mysql` exposes MySQL to the **whole internet even if `ufw deny 3306` is set**. Fixes: publish on localhost only (`-p 127.0.0.1:3306:3306`), don't publish database ports at all (use a Docker network), or put your rules in the **`DOCKER-USER`** chain, which Docker leaves alone:
> ```bash
> sudo iptables -I DOCKER-USER -i eth0 -p tcp --dport 3306 ! -s 10.0.11.0/24 -j DROP
> ```

### 24.8 nftables in 30 seconds

Modern distros use nftables underneath - `iptables` is often really `iptables-nft`, a compatibility layer (`iptables -V` shows `nf_tables`).

```bash
sudo nft list ruleset                                    # everything, all tables
sudo nft add rule inet filter input tcp dport 443 accept
```

You can keep using iptables syntax for years, but `nft list ruleset` is how you see what is *really* loaded.

### 24.9 ufw - the Ubuntu way

```bash
sudo ufw status verbose                 # is it on? defaults? rules?
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow OpenSSH                  # app profile (see: sudo ufw app list)
sudo ufw allow 443/tcp
sudo ufw allow 'Nginx Full'             # 80 + 443
sudo ufw allow from 10.0.11.0/24 to any port 5432 proto tcp
sudo ufw allow from 203.0.113.5 to any port 22
sudo ufw deny from 198.51.100.7         # block one bad IP
sudo ufw limit 22/tcp                   # rate-limit: blocks IPs with 6+ attempts in 30s

sudo ufw enable
sudo ufw status numbered
sudo ufw delete 3                       # delete by number
sudo ufw delete allow 443/tcp           # or by rule
sudo ufw logging on                     # logs go to /var/log/ufw.log
sudo ufw reset                          # wipe everything and start again
```

ufw rules are permanent automatically, and IPv6 rules are added for you (`IPV6=yes` in `/etc/default/ufw`).

### 24.10 firewalld - the RHEL way (zones)

firewalld groups rules into **zones** (trust levels). Each interface or source IP belongs to one zone.

| Zone | Trust | Typical use |
|------|-------|-------------|
| `drop` | Lowest - drop everything incoming, no reply | Hostile networks |
| `block` | Reject everything incoming | |
| `public` | **Default** - only selected services | Internet-facing interface |
| `internal` / `trusted` | High / everything allowed | Private LAN, VPN interface |

**Runtime vs permanent** - the #1 firewalld confusion:

- Without `--permanent`: the change works **now** and is lost on reload/reboot.
- With `--permanent`: saved to disk, but **not active until `--reload`**.

```bash
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all                          # rules in the default zone
sudo firewall-cmd --list-all --zone=internal

sudo firewall-cmd --permanent --add-service=http --add-service=https
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --remove-service=cockpit
sudo firewall-cmd --reload

# Trust the app subnet: everything from 10.0.11.0/24 uses the "internal" zone
sudo firewall-cmd --permanent --zone=internal --add-source=10.0.11.0/24

# Rich rule: SSH only from one IP
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="203.0.113.5/32" service name="ssh" accept'

# NAT gateway and port forwarding
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --permanent --add-forward-port=port=8080:proto=tcp:toport=80:toaddr=10.0.11.10

sudo firewall-cmd --reload

# Tested something at runtime and it works? Save it:
sudo firewall-cmd --runtime-to-permanent
```

### 24.11 Same job, three tools

| Task | iptables | ufw | firewalld |
|------|----------|-----|-----------|
| Allow HTTPS | `iptables -A INPUT -p tcp --dport 443 -j ACCEPT` | `ufw allow 443/tcp` | `firewall-cmd --permanent --add-service=https` |
| SSH from one IP | `iptables -A INPUT -p tcp -s 1.2.3.4 --dport 22 -j ACCEPT` | `ufw allow from 1.2.3.4 to any port 22` | rich rule (above) |
| Block an IP | `iptables -I INPUT -s 1.2.3.4 -j DROP` | `ufw deny from 1.2.3.4` | `firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=1.2.3.4 drop'` |
| List rules | `iptables -L -n -v` | `ufw status numbered` | `firewall-cmd --list-all` |
| Make permanent | `netfilter-persistent save` | automatic | `--permanent` + `--reload` |

### 24.12 Don't lock yourself out

```bash
# Schedule an automatic rollback in 5 minutes BEFORE you change remote firewall rules
sudo iptables-save > /root/fw-ok.rules
echo "iptables-restore < /root/fw-ok.rules" | sudo at now + 5 minutes

# ...apply your new rules, test SSH from a NEW terminal...
# Still connected? Cancel the rollback:
sudo atq            # find the job number
sudo atrm <job>
```

For ufw: `echo "ufw disable" | sudo at now + 5 minutes`. In the cloud you can also use the serial console, or simply rely on the **security group** as the main firewall and keep the host firewall as a second layer.

---

## 25. VPN

**What it is:** An encrypted tunnel across a public network, which also gives you an IP **inside** the private network.

![VPN](images/20-vpn.png)

### 25.1 The two types

| Type | Connects | Used for |
|------|----------|----------|
| **Remote access** | One person → company network | Engineers reaching private servers from home |
| **Site-to-site** | Whole office/data centre → cloud VPC | Hybrid infrastructure, permanent IPsec tunnel |

### 25.2 What it changes

| Without VPN | With VPN |
|-------------|----------|
| Private server `10.0.21.10` is unreachable | You get `10.8.0.6` and can `ssh 10.0.21.10` |
| Anyone on the café Wi-Fi sees your traffic metadata | Encrypted end to end |
| Admin panels need public exposure | Admin panels stay private - VPN only |

### 25.3 Tools you will meet

| Tool | Note |
|------|------|
| **WireGuard** | Modern, fast, tiny config. Current default choice |
| **OpenVPN** | Older, very compatible, heavier |
| **IPsec / IKEv2** | Standard for site-to-site between routers and clouds |
| **Tailscale / ZeroTier** | WireGuard with the hard parts (keys, NAT traversal) done for you |

```bash
# WireGuard
sudo wg-quick up wg0
sudo wg show                 # peers, handshakes, transfer
sudo wg-quick down wg0

# Prove you are inside the private network
ip a | grep wg0
ping 10.0.21.10
```

> 💡 **Tip:** The safest production pattern: **no public IPs on app or database servers**. Only the load balancer is public; humans get in through a VPN or a bastion host. If someone asks you to open a database port to `0.0.0.0/0`, suggest the VPN instead.

> ⚠️ **Warning:** If your home network is `192.168.1.0/24` and the office VPN is also `192.168.1.0/24`, routing breaks in confusing ways. That is why companies pick unusual ranges like `10.42.0.0/16`.

---

## 26. VPC: Your Private Network in the Cloud

**What it is:** A **VPC** (Virtual Private Cloud) is your own isolated network inside AWS, GCP or Azure. You choose its IP range, cut it into subnets, decide what can reach the internet, and put firewalls around everything. Every idea from this guide - CIDR, subnets, routing, NAT, gateways, firewalls, DNS, VPN - comes together here.

**Why DevOps people care:** Almost every cloud deployment starts with "create the VPC". Most "it can't connect" tickets in the cloud are a VPC problem: wrong route table, missing NAT gateway, security group, or NACL.

```mermaid
flowchart TB
    USERS(("🌍 Internet users")) --> IGW
    subgraph VPC["VPC 10.0.0.0/16"]
        IGW["Internet Gateway"]
        subgraph PUB["Public subnets - 10.0.1.0/24, 10.0.2.0/24<br/>route: 0.0.0.0/0 → IGW"]
            ALB["Application Load Balancer"]
            NAT["NAT Gateway<br/>(+ Elastic IP)"]
            BASTION["Bastion host"]
        end
        subgraph PRIV["Private subnets - 10.0.11.0/24, 10.0.12.0/24<br/>route: 0.0.0.0/0 → NAT GW"]
            APP1["App server AZ-a"]
            APP2["App server AZ-b"]
        end
        subgraph DATA["Data subnets - 10.0.21.0/24, 10.0.22.0/24<br/>no internet route"]
            DB[("Database<br/>primary + replica")]
        end
        IGW --> ALB
        ALB --> APP1 & APP2
        APP1 & APP2 --> DB
        APP1 & APP2 -.->|"apt update / API calls"| NAT
        NAT -.-> IGW
        EP["VPC Endpoint (S3)"]
        APP1 -.-> EP
    end
```

### 26.1 The building blocks

| Component | What it does | On-prem equivalent |
|-----------|--------------|-------------------|
| **VPC** | Your isolated network with a CIDR block (e.g. `10.0.0.0/16`) | The whole office network |
| **Subnet** | A slice of the VPC CIDR, locked to **one availability zone** | A VLAN |
| **Route table** | Decides where traffic from a subnet goes (Section 12.6) | Router config |
| **Internet Gateway (IGW)** | Two-way door between the VPC and the internet | The edge router |
| **NAT Gateway** | Outbound-only internet for private subnets (Section 10) | Home router NAT |
| **Elastic / static public IP** | A public IP that stays the same | The ISP's static IP |
| **Security Group** | Stateful firewall around each instance / ENI | Host firewall |
| **Network ACL (NACL)** | Stateless firewall around each subnet | Router ACL |
| **VPC Endpoint** | Reach cloud services (S3, DynamoDB, ECR…) privately, without the internet | - |
| **VPC Peering / Transit Gateway** | Connect VPCs together | WAN links |
| **VPN Gateway / Direct Connect** | Connect the VPC to the office / data centre | Site-to-site VPN / leased line |
| **Flow Logs** | Record of accepted / rejected connections | Firewall logs |
| **DNS resolver** | Built-in DNS at the VPC base address + 2 (e.g. `10.0.0.2`), private hosted zones | Internal DNS server |

### 26.2 What makes a subnet "public" or "private"

It is **only the route table**, nothing else:

| Subnet type | Route table has | Instances need | Reachable from internet? |
|-------------|----------------|----------------|-------------------------|
| **Public** | `0.0.0.0/0 → igw-…` | A public / Elastic IP | ✅ Yes (if the SG allows) |
| **Private** | `0.0.0.0/0 → nat-…` | Nothing | ❌ No, but can reach out |
| **Isolated / data** | Only `10.0.0.0/16 → local` | Nothing | ❌ No, and cannot reach out |

### 26.3 Security Groups vs NACLs

| | Security Group | Network ACL |
|---|---|---|
| Attached to | Instance / network interface | Subnet |
| State | **Stateful** - replies allowed automatically | **Stateless** - must allow the return traffic (ports `1024-65535`) |
| Rules | Allow only | Allow **and** deny |
| Order | All rules evaluated together | Numbered, lowest first, first match wins |
| Can reference | Other security groups (`sg-app` → `sg-db`) | Only CIDR ranges |
| Default | Deny all in, allow all out | Allow all (the default NACL) |
| Use it for | 99% of your rules | Subnet-wide blocks (e.g. ban an IP range) |

> 💡 **Tip:** Reference security groups instead of IPs: "the DB security group allows 5432 **from `sg-app`**". When app servers scale or change IP, the rule still works.

### 26.4 Follow one request through the VPC

```
User → https://app.example.com
 1. DNS (Route 53)         → ALB's public IP
 2. Internet → IGW         → ALB in the public subnet         SG-alb allows 443 from 0.0.0.0/0
 3. ALB → 10.0.11.10:8080  → app in the private subnet        SG-app allows 8080 from SG-alb
 4. App → 10.0.21.10:5432  → database in the data subnet      SG-db allows 5432 from SG-app
 5. App → api.stripe.com   → route 0.0.0.0/0 → NAT GW → IGW   outbound only
 6. App → S3               → VPC gateway endpoint             never leaves the AWS network
```

If any step breaks, you now know exactly which route table or security group to check.

### 26.5 Connecting networks together

| Option | Connects | Note |
|--------|----------|------|
| **VPC Peering** | VPC ↔ VPC (same or different account/region) | Simple and cheap, but **not transitive**: A↔B and B↔C does *not* give A↔C |
| **Transit Gateway** | Many VPCs + VPNs through one hub | The answer when you have more than a handful of VPCs |
| **Site-to-site VPN** | Office / data centre ↔ VPC | IPsec over the internet, up in minutes (Section 25) |
| **Direct Connect / ExpressRoute / Interconnect** | Data centre ↔ cloud over a private line | Stable latency, high bandwidth, takes weeks to set up |
| **PrivateLink / Interface endpoint** | Expose **one service** to another VPC | No CIDR overlap issues, no full network access |

> ⚠️ **Warning:** Peering and Transit Gateway **require non-overlapping CIDRs**. If every team created `10.0.0.0/16`, none of those VPCs can ever be connected. Plan an IP address map for the whole company up front (e.g. `10.10.0.0/16` prod, `10.20.0.0/16` staging, `10.30.0.0/16` dev).

### 26.6 Same thing, different names

| Concept | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Private network | VPC | VNet | VPC (global, subnets are regional) |
| Instance firewall | Security Group | NSG | Firewall rules (by tag / service account) |
| Subnet firewall | NACL | NSG on subnet | - |
| Outbound NAT | NAT Gateway | NAT Gateway | Cloud NAT |
| Connect networks | Peering, Transit Gateway | VNet Peering, Virtual WAN | VPC Peering, Shared VPC, NCC |
| Private service access | VPC Endpoint / PrivateLink | Private Endpoint | Private Service Connect |
| Private line | Direct Connect | ExpressRoute | Cloud Interconnect |
| Traffic logs | VPC Flow Logs | NSG Flow Logs | VPC Flow Logs |

### 26.7 A VPC in Terraform (the shape of it)

```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
}

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-southeast-2a"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.11.0/24"
  availability_zone = "ap-southeast-2a"
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_a.id      # NAT GW lives in a PUBLIC subnet
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
}

resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.private.id
}
```

In real projects, most teams use the community module [`terraform-aws-modules/vpc/aws`](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest), which builds all of this from a few lines.

### 26.8 Inspect a VPC from the CLI

```bash
aws ec2 describe-vpcs --query "Vpcs[].[VpcId,CidrBlock]" --output table
aws ec2 describe-subnets --filters Name=vpc-id,Values=vpc-0abc --query "Subnets[].[SubnetId,CidrBlock,AvailabilityZone]" --output table
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-0abc
aws ec2 describe-security-groups --group-ids sg-0abc
aws ec2 describe-nat-gateways

# From inside an instance - who am I, where am I? (IMDSv2)
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/local-ipv4
```

**VPC Flow Logs** answer "was this connection allowed or rejected?":

```
2 123456789012 eni-0abc 203.0.113.5 10.0.11.10 51234 22 6 10 840 1700000000 1700000060 REJECT OK
```

Read it as: from `203.0.113.5:51234` to `10.0.11.10` port `22`, protocol `6` (TCP), **REJECT** → a security group or NACL blocked this SSH attempt.

### 26.9 Common VPC mistakes

| Symptom | Cause |
|---------|-------|
| Instance in a "public" subnet unreachable | No public IP, or the route table has no `0.0.0.0/0 → IGW` |
| Private instance cannot `apt update` / pull images | No NAT gateway, NAT GW placed in a private subnet, or route missing |
| Works in AZ-a, broken in AZ-b | Second subnet uses a different route table (or the NAT GW only exists in AZ-a) |
| SG allows it, still blocked | The **NACL** is blocking, often the return ports `1024-65535` |
| Can't peer two VPCs | Overlapping CIDRs |
| A↔B and B↔C work, A↔C doesn't | Peering is not transitive - use a Transit Gateway |
| Huge NAT gateway bill | Heavy S3/ECR traffic going through NAT - add VPC endpoints |
| Subnet ran out of IPs | `/24` minus 5 reserved, and EKS gives every pod a VPC IP - size subnets generously |

---

# PART 4: Tools and Practice

---

## 27. The Networking Toolbox

Every tool answers one question at one layer. Learn which question each one answers and troubleshooting becomes mechanical.

![Tools by layer](images/21-tools-by-layer.png)

### 27.1 `ping` - can I reach the host at all?

```bash
ping example.com            # Linux/Mac: runs until Ctrl+C
ping -c 4 example.com       # stop after 4
ping -n 4 example.com       # Windows
```

```
64 bytes from 93.184.216.34: icmp_seq=1 ttl=56 time=12.4 ms
--- 4 packets transmitted, 4 received, 0% packet loss
rtt min/avg/max/mdev = 12.1/12.6/13.2/0.4 ms
```

| What you read | What it tells you |
|---------------|-------------------|
| `time=12.4 ms` | Latency. Same city ~1-20ms, another continent ~150-300ms |
| `0% packet loss` | Healthy. Any loss above 0% on a wired link is a problem |
| `ttl=56` | Hops left before the packet dies - roughly how far away it is |
| `Request timeout` | Host down, **or** ICMP is simply blocked (very common in clouds) |

> ⚠️ **Warning:** "Ping fails" does **not** mean "the server is down". Many clouds block ICMP by default. Always confirm with `nc -zv host port` before declaring an outage.

### 27.2 `traceroute` / `tracert` / `mtr` - where does it break?

```bash
traceroute example.com         # Linux/Mac
traceroute -T -p 443 host      # use TCP instead of UDP (gets through more firewalls)
tracert example.com            # Windows
mtr example.com                # live, continuously updating - the best of the three
```

```
1  192.168.1.1      1.2 ms      ← your router
2  10.20.30.1      11.8 ms      ← your ISP
3  * * *                        ← this hop hides itself (often fine)
8  93.184.216.34   12.4 ms      ← destination
```

Read it as: latency should climb gradually. A jump from 12ms to 300ms at one hop shows you where the slowness starts. `* * *` at a single hop is usually just a router that ignores traceroute; `* * *` all the way to the end means the path really is broken.

### 27.3 `dig` / `nslookup` - is the name right?

```bash
dig +short app.example.com        # the answer, nothing else
dig app.example.com               # with TTL and sections
dig @1.1.1.1 app.example.com      # ask a specific resolver
dig +trace app.example.com        # full path from the root servers
dig -x 93.184.216.34              # reverse lookup
nslookup app.example.com          # the Windows-friendly option
```

### 27.4 `curl` - is the application answering?

```bash
curl -I https://example.com                      # headers only
curl -v https://example.com                      # handshake + headers + body
curl -L https://example.com                      # follow redirects
curl -s -o /dev/null -w "%{http_code}\n" URL     # just the status code
curl -w "dns:%{time_namelookup} tls:%{time_appconnect} total:%{time_total}\n" -o /dev/null -s URL
curl --resolve app.example.com:443:203.0.113.50 https://app.example.com   # test a server before DNS
curl -x http://proxy:3128 https://example.com    # through a proxy
```

### 27.5 `wget` - download files and mirror sites

```bash
wget https://example.com/file.tar.gz
wget -c https://example.com/big.iso          # continue a broken download
wget -q --spider https://example.com && echo UP    # just check it is alive
wget -r -l 2 https://example.com             # recursive download (be polite)
```

| | `curl` | `wget` |
|---|---|---|
| Best at | APIs, headers, debugging | Downloading files, retries, recursion |
| Output | stdout by default | A file by default |
| In scripts | Health checks, API calls | Fetching artifacts |

### 27.6 `ss` and `netstat` - what is listening here?

```bash
ss -tulpn                  # TCP+UDP listening, with process names
ss -tan                    # all TCP sockets and their states
ss -tn state established   # current established connections
ss -s                      # summary counts

netstat -tulpn             # older equivalent
netstat -an | findstr LISTENING    # Windows
```

TCP states you will see: `LISTEN` (waiting), `ESTABLISHED` (connected), `TIME_WAIT` (recently closed - harmless in small numbers), `SYN_SENT` (handshake never answered → firewall).

### 27.7 `telnet` and `nc` - is the port open?

```bash
nc -zv example.com 443            # preferred
nc -zv -w 3 10.0.1.11 3306        # with a 3 second timeout
telnet example.com 443            # old but available everywhere
```

```
Connection to example.com 443 port [tcp/https] succeeded!   ✅ open
nc: connect to ... Connection refused                        ❌ nothing listening
nc: connect to ... Operation timed out                       🔥 firewall dropping
```

You can even speak HTTP by hand:

```bash
telnet example.com 80
GET / HTTP/1.1
Host: example.com
          ← press Enter twice
```

**netcat is a Swiss army knife**, not just a port checker:

```bash
# Scan a small range of ports
nc -zv -w 1 10.0.1.11 20-25

# Check a UDP port (DNS) - UDP gives weaker answers, "succeeded" only means no rejection
nc -zvu 8.8.8.8 53

# Start a tiny listener - prove a firewall path works BEFORE the real app exists
nc -l 9000                 # on the server (some versions: nc -l -p 9000)
nc SERVER_IP 9000          # on the client - type, and it appears on the server

# Copy a file without SSH/SCP (inside a trusted network only - it is not encrypted)
nc -l 9000 > backup.tar.gz                # receiver
nc -N SERVER_IP 9000 < backup.tar.gz      # sender (-N closes when done)

# Fake an HTTP server in one line, to test a load balancer health check
while true; do printf 'HTTP/1.1 200 OK\r\nContent-Length: 2\r\n\r\nOK' | nc -l 8080; done
```

> 💡 **Tip:** The listener trick is the cleanest way to split a problem in half: if `nc` can connect to your `nc -l`, the network and firewall are fine and the problem is the application.

### 27.8 `ip` - addresses, routes and links

```bash
ip a                  # addresses
ip link               # interfaces up/down
ip route              # routing table, including "default via" (your gateway)
ip neigh              # ARP table: IP → MAC on this LAN
```

### 27.9 `tcpdump` - see the actual packets

```bash
sudo tcpdump -i any -n port 443
sudo tcpdump -i any -n host 10.0.1.11
sudo tcpdump -i any -n port 53                 # watch DNS queries live
sudo tcpdump -i any -n -w capture.pcap         # save for Wireshark
```

Useful filters and flags:

```bash
sudo tcpdump -D                                        # list interfaces
sudo tcpdump -i eth0 -n -c 20 port 80                  # stop after 20 packets
sudo tcpdump -i any -n src 10.0.1.11 and dst port 5432 # combine with and / or / not
sudo tcpdump -i any -n net 10.0.11.0/24                # a whole subnet
sudo tcpdump -i any -n 'tcp[tcpflags] & (tcp-syn|tcp-rst) != 0'   # only SYNs and RSTs
sudo tcpdump -i any -n -A port 80                      # print payload as text (plain HTTP)
sudo tcpdump -i any -n icmp                            # watch ping / traceroute
sudo tcpdump -r capture.pcap -n                        # read a saved capture
```

Reading one line:

```
10:15:02.114 IP 10.0.1.25.51544 > 93.184.216.34.443: Flags [S], seq 123, win 64240, length 0
              └─ source IP.port   └─ destination IP.port  └─ S = SYN
```

| Flag | Meaning | What it tells you |
|------|---------|-------------------|
| `[S]` | SYN | Client trying to connect |
| `[S.]` | SYN-ACK | Server answered - the port is open |
| `[.]` | ACK | Normal traffic |
| `[P.]` | PUSH + ACK | Data being sent |
| `[F.]` | FIN | Polite close |
| `[R]` / `[R.]` | RST | Refused or killed - nothing listening, or a firewall REJECT |

**Only `[S]` repeating with no `[S.]`** = the SYN never gets an answer → firewall drop or routing problem. **`[S]` followed by `[R.]`** = the host is reachable but nothing listens on that port.

Use it when every other tool says "should be fine" and it still is not.

### 27.10 One-line reference

| Tool | Answers |
|------|---------|
| `ping` | Is the host reachable, and how far away? |
| `traceroute` / `mtr` | Where on the path does it break or slow down? |
| `dig` / `nslookup` | Does the name resolve, and to what? |
| `curl` | Does the application reply, with what status and timing? |
| `wget` | Can I download this file? |
| `ss` / `netstat` | What is listening on this machine, and to whom? |
| `nc` / `telnet` | Is that port open from here? |
| `ip` / `ifconfig` | What are my addresses, routes and gateway? |
| `tcpdump` | What is actually on the wire? |

---

## 28. Troubleshooting Playbook

![Troubleshooting flow](images/22-troubleshooting-flow.png)

### 28.1 "The site is down" - run these in order

```bash
# 1. Does the name resolve, and to the IP you expect?
dig +short app.example.com

# 2. Is the host reachable? (ICMP may be blocked - don't panic yet)
ping -c 3 203.0.113.50
traceroute 203.0.113.50

# 3. Is the port open from here?
nc -zv app.example.com 443

# 4. On the server: is the app really listening, and on which address?
ss -tulpn | grep -E ':(80|443|8080)'

# 5. What does HTTP say?
curl -v https://app.example.com
```

### 28.2 Symptom → likely cause

| Symptom | Most likely cause | Check |
|---------|------------------|-------|
| `NXDOMAIN` / "could not resolve host" | DNS record missing or typo'd | `dig`, `/etc/resolv.conf` |
| Resolves to the **old** IP | TTL cache | `dig @8.8.8.8`, lower TTL |
| `Connection refused` | Service down or wrong port | `ss -tulpn`, `systemctl status` |
| `Connection timed out` | Firewall / security group dropping | Firewall rules, routes |
| Works locally, not remotely | App bound to `127.0.0.1` | `ss -tulpn` → change to `0.0.0.0` |
| `502 Bad Gateway` | Backend down or wrong `proxy_pass` port | Backend logs, `curl localhost:3000` |
| `503 Service Unavailable` | No healthy LB targets | Health check path/port, SG between LB and app |
| `504 Gateway Timeout` | Backend too slow | App timeouts, slow queries |
| TLS certificate error | Expired, wrong name, missing chain | `openssl s_client` |
| Private server cannot `apt update` | Missing NAT gateway or route | Route table, NAT gateway |
| Works on IPv4, breaks sometimes | Broken `AAAA` record | `dig AAAA`, `curl -4` |
| Intermittent slowness | Packet loss on the path | `mtr` for a few minutes |
| `Network is unreachable` | No matching route / no default gateway | `ip route`, `ip route get <ip>` |
| Works one direction, hangs the other | Asymmetric routing, or NACL missing return ports | Route tables on both sides, NACL `1024-65535` |
| Docker port open to the world despite `ufw deny` | Docker's iptables rules bypass ufw | Bind to `127.0.0.1`, use `DOCKER-USER` chain |
| Firewall rule "added" but no effect | Rule placed after an earlier DROP, or firewalld `--permanent` without `--reload` | `iptables -L --line-numbers`, `firewall-cmd --list-all` |
| Works in Chrome, fails in curl/Java | Missing intermediate certificate | Serve `fullchain.pem`, test with `openssl s_client` |

### 28.3 The two habits that matter

1. **Change one thing at a time.** Then re-test. Three simultaneous changes make the cause unfindable.
2. **Prove each layer before moving up.** Name → host → port → application. Do not debug HTTP headers when the port is closed.

> 💡 **Tip:** Write down what you tested and what the result was while you work. When you hand it to a colleague or write the incident report, you already have the timeline.

---

## 29. Practice Labs

Reading is not learning. Do these on your laptop or a free cloud VM.

![Practice labs](images/23-practice-labs.png)

### Lab 1: Map your own network (15 minutes)

```bash
ip a                        # your private IP and mask
ip route                    # your gateway
cat /etc/resolv.conf        # your DNS resolver
curl ifconfig.me            # your public IP
traceroute google.com       # the path out
```

**Write down:** your IP and CIDR, the network address, the broadcast address, how many usable hosts your network has, and how many hops to Google.

### Lab 2: Subnet a VPC on paper (20 minutes)

Split `10.0.0.0/16` for 3 availability zones with one public and one private subnet each (6 subnets). Give each a `/24`. Write the network, usable range and broadcast for every one. Check yourself with `ipcalc`.

### Lab 3: Break and fix DNS on purpose (15 minutes)

```bash
sudo sh -c 'echo "127.0.0.1 example.com" >> /etc/hosts'
curl -I http://example.com      # now fails or hits your own machine
dig +short example.com          # dig ignores /etc/hosts - still the real IP!
```

That difference between `dig` and `curl` is the lesson. Then remove the line you added.

### Lab 4: Reverse proxy two apps (45 minutes)

```bash
mkdir app && echo "<h1>APP</h1>" > app/index.html && (cd app && python3 -m http.server 3000 &)
mkdir api && echo '{"ok":true}'  > api/index.html && (cd api && python3 -m http.server 5000 &)
```

Install nginx, route `/` → `:3000` and `/api/` → `:5000`, then `curl localhost/` and `curl localhost/api/`. Stop the app on 3000 and watch nginx return **502** - that is the error you will meet in production.

### Lab 5: Lock a server down (30 minutes)

On a cloud VM: default deny, allow 443 from anywhere, allow 22 **only** from your current IP (`curl ifconfig.me`). Test from a phone hotspot - SSH should time out, HTTPS should work. Keep a second SSH session open while you do this.

### Lab 6: Debug a dead port (20 minutes)

```bash
python3 -m http.server 8000 --bind 127.0.0.1     # bound to localhost only
```

From another machine: `nc -zv SERVER_IP 8000` → refused/timeout. Prove why with `ss -tulpn | grep 8000`, then restart with `--bind 0.0.0.0` and test again.

### Lab 7: Read a TLS certificate (10 minutes)

```bash
echo | openssl s_client -connect github.com:443 -servername github.com 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

Who issued it, which names it covers, and when it expires. Now you can check an "is the certificate expired?" incident in 10 seconds.

### Lab 8: Watch DNS traffic live (10 minutes)

```bash
sudo tcpdump -i any -n port 53        # terminal 1
dig github.com                        # terminal 2
```

You will see the query leave and the answer come back - DNS stops being magic.

### Lab 9: Read and change your routes (15 minutes)

```bash
ip route                         # find your default gateway
ip route get 8.8.8.8             # which route and interface would be used
ip route get 192.168.1.1
traceroute -n 8.8.8.8            # the first hop should be your gateway
```

On a test VM (not your only connection!): `sudo ip route del default`, then `ping 8.8.8.8` → "Network is unreachable". Put it back with `sudo ip route add default via <gateway>`. You have just seen what "no default route" looks like.

### Lab 10: Build an iptables firewall and watch the counters (30 minutes)

On a test VM, run the rule set from **24.3** (with your own IP in the SSH rule), then:

```bash
sudo iptables -L INPUT -n -v --line-numbers     # note the pkts counters
curl -I http://SERVER_IP                         # from your laptop - counter on the 80/443 rule goes up
nc -zv -w 3 SERVER_IP 3306                       # times out (DROP)
sudo journalctl -k | grep IPT-DROP               # the dropped attempt is logged
```

Now append `sudo iptables -A INPUT -p tcp -j REJECT --reject-with tcp-reset` and repeat the `nc` test: "timed out" becomes "refused". That is DROP vs REJECT, felt from the outside. Save with `netfilter-persistent save` and reboot to prove the rules survive.

### Lab 11: HTTPS with your own certificate (30 minutes)

```bash
openssl req -x509 -newkey rsa:2048 -nodes -days 30 \
  -keyout server.key -out server.crt -subj "/CN=myapp.local" \
  -addext "subjectAltName=DNS:myapp.local"
```

Configure nginx with `listen 443 ssl` and these two files, add `127.0.0.1 myapp.local` to `/etc/hosts`, then:

```bash
curl https://myapp.local                      # fails: self signed certificate
curl --cacert server.crt https://myapp.local  # works: you told curl to trust it
openssl s_client -connect myapp.local:443 -servername myapp.local </dev/null | head -20
curl -v https://myapp.local 2>&1 | grep -E "SSL connection|TLSv"   # see the TLS version and cipher
```

### Lab 12: Draw (or build) a VPC (45 minutes)

On paper, or in a free-tier AWS account with the Terraform from **26.7**: one VPC, a public and a private subnet, an IGW, a NAT gateway and two route tables. Launch one instance in each subnet. Prove that:

1. You can SSH to the public instance, but not directly to the private one.
2. From the public instance you can SSH to the private one (bastion pattern).
3. The private instance can `curl https://example.com` (through NAT), and `curl ifconfig.me` shows the **NAT gateway's** Elastic IP.

Then **delete everything** - NAT gateways cost money every hour.

---

## 30. Cheat Sheet

```bash
# ───── Identity ─────────────────────────────────────────────
ip a / ipconfig              # my addresses
ip route                     # my gateway
curl ifconfig.me             # my public IP
hostname -I                  # just the IPs

# ───── Name resolution ──────────────────────────────────────
dig +short host              # name → IP
dig MX|TXT|NS domain         # other records
dig @8.8.8.8 host            # ask a specific resolver
nslookup host                # Windows-friendly

# ───── Reachability ─────────────────────────────────────────
ping -c 4 host               # is it alive (if ICMP allowed)
traceroute host              # path and where it breaks
mtr host                     # live traceroute + loss

# ───── Ports ────────────────────────────────────────────────
nc -zv host 443              # is the port open from here
ss -tulpn                    # what is listening locally
sudo lsof -i :8080           # who owns this port

# ───── HTTP ─────────────────────────────────────────────────
curl -I url                  # headers
curl -v url                  # full conversation
curl -s -o /dev/null -w "%{http_code}\n" url
curl --resolve host:443:IP https://host      # test before DNS cutover

# ───── TLS ──────────────────────────────────────────────────
echo | openssl s_client -connect host:443 -servername host 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates

# ───── Packets ──────────────────────────────────────────────
sudo tcpdump -i any -n port 443
sudo tcpdump -i any -n -w capture.pcap         # save for Wireshark

# ───── Routing ──────────────────────────────────────────────
ip route                     # route table + default gateway
ip route get 8.8.8.8         # which route would be used
sudo ip route add 10.20.0.0/16 via 192.168.1.254
sysctl net.ipv4.ip_forward   # is this box allowed to route?

# ───── Firewall ─────────────────────────────────────────────
sudo iptables -L -n -v --line-numbers
sudo iptables -t nat -L -n -v
sudo iptables-save > backup.rules
sudo ufw status numbered
sudo firewall-cmd --list-all

# ───── Certificates ─────────────────────────────────────────
openssl x509 -in cert.pem -noout -subject -issuer -dates -ext subjectAltName
sudo certbot certificates
sudo certbot renew --dry-run

# ───── netcat ───────────────────────────────────────────────
nc -l 9000                   # listen (test a firewall path)
nc host 9000                 # connect to it
```

### Numbers worth memorising

| Thing | Value |
|-------|-------|
| Private ranges | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` |
| Usable hosts | `2^(32 - prefix) - 2` |
| `/24` | 254 usable &nbsp;·&nbsp; `/26` → 62 &nbsp;·&nbsp; `/30` → 2 |
| Loopback | `127.0.0.1` |
| "Everything" | `0.0.0.0/0` &nbsp;·&nbsp; "one host" → `/32` |
| DHCP failed | `169.254.x.x` |
| Core ports | 22 SSH · 53 DNS · 80 HTTP · 443 HTTPS · 3306 MySQL · 5432 Postgres · 6379 Redis |
| Refused vs timeout | Refused = service problem · Timeout = firewall problem |
| AWS reserved IPs | 5 per subnet → `/24` = 251 usable |
| Let's Encrypt validity | 90 days (renewed around day 60) |
| Modern TLS | 1.2 and 1.3 only |
| Route choice | Longest prefix wins, then lowest metric |
| Public subnet | Route `0.0.0.0/0 → Internet Gateway` |

---

## 31. Interview Questions

Try to answer out loud before you open each one.

<details>
<summary><b>1. What is the difference between the OSI and TCP/IP models?</b></summary>

OSI is a 7-layer reference model used for vocabulary and teaching. TCP/IP is the 4-layer model the internet actually runs on (Application, Transport, Internet, Link). OSI's application, presentation and session layers are merged into TCP/IP's application layer. See Sections 2-3.
</details>

<details>
<summary><b>2. What happens when you type a URL into the browser and press Enter?</b></summary>

DNS lookup (browser cache → OS → resolver → root → TLD → authoritative), TCP 3-way handshake to port 443, TLS handshake (certificate validation, key exchange), HTTP request, server (often via load balancer / reverse proxy) returns a response, browser renders and fetches more resources. See Section 5.
</details>

<details>
<summary><b>3. TCP vs UDP - when would you use each?</b></summary>

TCP is connection-oriented, reliable and ordered (HTTP, SSH, databases). UDP is connectionless and fast with no delivery guarantee (DNS, DHCP, video, VoIP, metrics). Use UDP when late data is useless anyway. See Section 13.
</details>

<details>
<summary><b>4. How many usable hosts are in a /26? Split 10.0.0.0/24 into 4 subnets.</b></summary>

`2^(32-26) - 2 = 62`. Four `/26` subnets: `10.0.0.0/26`, `10.0.0.64/26`, `10.0.0.128/26`, `10.0.0.192/26`. (In AWS, 59 usable each because 5 are reserved.) See Sections 7-8.
</details>

<details>
<summary><b>5. What are the private IP ranges, and why do they exist?</b></summary>

`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` (RFC 1918). They are not routed on the internet, so every organisation can reuse them; NAT translates them to public IPs for outbound traffic. See Section 9.
</details>

<details>
<summary><b>6. What does a default gateway do? How does a host decide where to send a packet?</b></summary>

If the destination is in the host's own subnet, it delivers directly (ARP for the MAC). Otherwise it sends it to the default gateway. Routers pick routes by longest prefix match, then lowest metric. See Section 12.
</details>

<details>
<summary><b>7. "Connection refused" vs "connection timed out" - what does each tell you?</b></summary>

Refused: the packet reached the host but nothing is listening on that port (or a firewall REJECTs) → service problem. Timed out: no reply at all → a firewall is DROPping, or there is no route. See Sections 13 and 23.
</details>

<details>
<summary><b>8. Explain the TLS handshake.</b></summary>

ClientHello (versions, ciphers, key share, SNI) → ServerHello (chosen cipher, key share) → both derive session keys with ECDHE → server sends certificate and proves it owns the private key → client validates the chain, name and dates → Finished → encrypted application data. TLS 1.3 does this in one round trip. See Section 17.
</details>

<details>
<summary><b>9. A site works in the browser but curl says "unable to get local issuer certificate". Why?</b></summary>

The server is not sending the intermediate certificate. Browsers often fill the gap; curl does not. Configure the server with `fullchain.pem`. See Section 17.3.
</details>

<details>
<summary><b>10. L4 vs L7 load balancer?</b></summary>

L4 routes on IP + port and forwards TCP/UDP without reading it (fast, any protocol, e.g. AWS NLB). L7 reads HTTP, so it can route by path/host/header, terminate TLS and do HTTP health checks (e.g. AWS ALB, nginx). See Section 21.
</details>

<details>
<summary><b>11. Forward proxy vs reverse proxy?</b></summary>

A forward proxy sits in front of clients and makes requests for them (corporate egress, `HTTP_PROXY`). A reverse proxy sits in front of servers and receives requests for them (nginx, load balancers, CDNs). See Section 22.
</details>

<details>
<summary><b>12. You get a 502 from nginx. How do you debug it?</b></summary>

nginx is up but the backend isn't answering. Check the backend process (`systemctl status`), that it listens on the `proxy_pass` port and an address nginx can reach (`ss -tulpn`), test it directly (`curl localhost:3000`), then read the nginx error log. See Sections 16 and 22.
</details>

<details>
<summary><b>13. Security group vs NACL?</b></summary>

Security groups are stateful, allow-only, attached to instances, and can reference other SGs. NACLs are stateless, allow + deny, attached to subnets, evaluated in number order, and need the return ephemeral ports allowed. See Section 26.3.
</details>

<details>
<summary><b>14. What makes a subnet public? How does a private instance reach the internet?</b></summary>

A public subnet's route table has `0.0.0.0/0 → Internet Gateway`. A private instance uses a route `0.0.0.0/0 → NAT Gateway`, and the NAT gateway sits in a public subnet. See Section 26.2.
</details>

<details>
<summary><b>15. iptables: DROP vs REJECT, and what does the ESTABLISHED,RELATED rule do?</b></summary>

DROP silently discards (client sees a timeout); REJECT sends back an error (client sees refused). `-m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT` allows replies to connections that were already permitted, which makes the firewall stateful. See Section 24.
</details>

<details>
<summary><b>16. You ran `ufw deny 3306`, yet a Docker MySQL container is reachable from the internet. Why?</b></summary>

Docker inserts its own iptables NAT/forward rules for published ports, and they are processed before ufw's rules. Bind to `127.0.0.1:3306:3306`, don't publish the port, or filter in the `DOCKER-USER` chain. See Section 24.7.
</details>

<details>
<summary><b>17. You changed a DNS record but users still reach the old server. Why, and how do you avoid it?</b></summary>

Resolvers cache the old answer for the record's TTL. Lower the TTL (e.g. to 60s) a day before the migration, switch, then raise it again. Check with `dig @8.8.8.8` and `dig +trace`. See Section 15.
</details>

<details>
<summary><b>18. Ping works, but the website doesn't load. What do you check?</b></summary>

ICMP working only proves layer 3. Check the port (`nc -zv host 443`), the listening address (`ss -tulpn`), the firewall / SG for that port, then HTTP and TLS with `curl -v`. If it hangs midway on big responses, suspect MTU (especially over a VPN). See Sections 4 and 28.
</details>

<details>
<summary><b>19. Why can't two VPCs with 10.0.0.0/16 be peered, and how do you prevent it?</b></summary>

Routes would be ambiguous - both sides claim the same addresses. Plan a non-overlapping IP address map per environment/account up front; for single services, PrivateLink works even with overlaps. See Section 26.5.
</details>

<details>
<summary><b>20. What is SNI and why does it matter?</b></summary>

Server Name Indication: the client sends the hostname in the ClientHello so a server with one IP can present the right certificate for many domains. Without it, you get the default certificate and a name mismatch. See Section 17.7.
</details>

---

## 32. Reference

- 📘 [Cloudflare Learning Center](https://www.cloudflare.com/learning/) - the clearest free explanations of DNS, TLS, proxies and CDNs
- 📘 [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/) - how sockets really work
- 📘 [High Performance Browser Networking](https://hpbn.co/) - free book, excellent on TCP, TLS and HTTP/2
- 🔧 [ipcalc online](https://jodies.de/ipcalc) - subnet calculator
- 🔧 [MDN HTTP docs](https://developer.mozilla.org/en-US/docs/Web/HTTP) - methods, headers, status codes
- 🔧 [curl cheat sheet](https://curl.se/docs/manpage.html) - every flag explained
- 🔧 [dig HOWTO](https://www.isc.org/bind/) - BIND and DNS tooling
- 🔒 [SSL Labs server test](https://www.ssllabs.com/ssltest/) - grade your TLS setup
- 🔒 [Mozilla SSL Config Generator](https://ssl-config.mozilla.org/) - nginx/Apache TLS config that is actually correct
- ☁️ [AWS VPC docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) - subnets, route tables, NAT, security groups
- ☸️ [Kubernetes networking](https://kubernetes.io/docs/concepts/services-networking/) - services, DNS and ingress
- 🦈 [Wireshark](https://www.wireshark.org/) - when `tcpdump` output needs a UI
- 🔥 [iptables tutorial (Frozentux)](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html) - the classic deep dive into chains and tables
- 🔥 [Ubuntu ufw guide](https://help.ubuntu.com/community/UFW) and [firewalld docs](https://firewalld.org/documentation/)
- 🔒 [Let's Encrypt - how it works](https://letsencrypt.org/how-it-works/) and [certbot](https://certbot.eff.org/)
- 🔒 [The Illustrated TLS 1.3 Connection](https://tls13.xargs.org/) - every byte of the handshake explained
- ☁️ [terraform-aws-modules/vpc](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) - production VPC in a few lines

> 💡 **Tip:** Learn by doing. Run every command in this guide on a real machine, break something on purpose, then fix it. That is the only way networking sticks.

⭐ If this guide helped you, give the repo a star!
