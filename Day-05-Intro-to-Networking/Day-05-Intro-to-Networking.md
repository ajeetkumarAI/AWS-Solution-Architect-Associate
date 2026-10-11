# Day 5 – Introduction to Networking

**Today's agenda**
- What is a network, and what is networking?
- Network devices
- IP addressing

---

## 0. Why networking, in an AWS course?

AWS services fall into a few big groups:

1. **Compute**
2. **Networking** ← we start here
3. **Identity**
4. **Storage & Database**
5. **Others**

Every AWS resource (EC2 server, database, load balancer) talks to others over a **network**. Later you will build your own private network in AWS called a **VPC**. To design one, you need networking basics first: networks, devices and IP addresses.

---

## 1. Level 1 – Basics: Network vs Networking

> **Network:** A group of computers **connected** together using media, **wired** (cables) or **wireless** (Wi-Fi).
>
> **Networking:** A group of computers **communicating** with each other through a network.

**Simple way to remember**
- **Network** = the *roads* (the connection)
- **Networking** = the *traffic* (the communication happening on the roads)

### From two PCs to many

```
 Two PCs, connected directly           Many PCs, through a central network

   PC1 ─────────── PC2                    PC1          PC2
   [NIC]           [NIC]                     \        /
                                              Network
                                             /        \
                                          PC3          PC4
```

Every PC needs a **NIC (Network Interface Card)** to join a network. It's the "socket" where the cable or Wi-Fi signal connects.

In real life the "Network" box in the middle is usually a **switch**:

```
   PC0 ──┐           ┌── PC1
         ├── Switch ─┤
   PC2 ──┘           └── PC3
```

---

## 2. Types of networks (by size)

| Type | Full form | Covers | Example |
|---|---|---|---|
| **PAN** | Personal Area Network | A few metres around one person | Phone + Bluetooth earbuds + smartwatch |
| **LAN** | Local Area Network | A limited area: room, home, office floor | PCs in one office connected by a switch |
| **CAN** | Campus Area Network | Several LANs across one campus | College buildings, a company tech park |
| **MAN** | Metropolitan Area Network | LANs of the same organization across a **city** | A bank connecting all its branches in Pune |
| **WAN** | Wide Area Network | LANs connected **globally** | A company linking Mumbai, London and New York offices |

```
 PAN  <  LAN  <  CAN  <  MAN  <  WAN
 (person) (room) (campus) (city) (world)
```

> ⚠️ **Small correction to class notes:** The notes list CAN and PAN as "types of LAN". It's clearer to treat them as **separate sizes**: PAN is *smaller* than a LAN, and CAN is *bigger* (it joins several LANs).
>
> The notes also call the internet "an application that works in WAN". More precisely, **the internet is the world's largest WAN**: a global network of networks. Web browsing, email and video calls are the applications that run *on* it.

---

## 3. Level 2 – Going deeper: Network devices

| Device | What it does | Real-life comparison |
|---|---|---|
| **NIC** (Network Interface Card) | Connects a single device to a network. Has a fixed hardware address (**MAC address**) | The door of your house |
| **Hub** | Connects many devices; sends every message to **all** of them (old, slow, insecure) | A loudspeaker announcement to everyone |
| **Switch** | Connects many devices in a LAN; sends each message **only to the right device** (uses MAC addresses) | A receptionist who hands each letter to the right person |
| **Router** | Connects **different networks** together, e.g. your LAN to the internet (uses IP addresses) | A post office sending letters between cities |
| **Wi-Fi router** | Router + switch + wireless access point in one box (your home router) | All-in-one post office at home |
| **Access point** | Adds wireless (Wi-Fi) access to an existing wired network | A wireless extension of the network |
| **Firewall** | Checks traffic and **allows or blocks** it based on rules | Security guard at the gate |

### Traditional IT infrastructure (the Packet Tracer diagram from class)

```
 ISP cloud ── Firewall ── Router ── Switch ──┬── PC0, PC1, PC2
 (Internet)   (guard)    (between   (inside  ├── Server
                          networks)  LAN)    └── Wi-Fi Router ── Laptop0, Laptop1
```

Traffic from the internet comes in from the **ISP**, passes the **firewall**, the **router** sends it into the office network, and the **switch** delivers it to the right PC, server or Wi-Fi device.

**Hub vs Switch vs Router (exam favourite)**

| | Hub | Switch | Router |
|---|---|---|---|
| Works inside | One LAN | One LAN | Between networks |
| Sends data to | Everyone | Only the right device | The right network |
| Uses | Nothing (just repeats) | MAC address | IP address |

---

## 4. IP addressing – the basics

### Communication needs rules

```
   John ───── basic communication (sign language) ───── Lina
   PC1  ─────────────────── IP ─────────────────────── PC2
 192.168.1.1                                        192.168.1.2
```

Two people can only talk if they share a language. Two computers can only talk if they follow the same **protocol**.

| Term | Meaning |
|---|---|
| **Protocol** | A set of rules for good communication (like grammar for computers) |
| **IP (Internet Protocol)** | The basic protocol that moves data from one device to another using addresses |
| **IP address** | A **32-bit (IPv4)** or **128-bit (IPv6)** **logical** address given to a PC or device to identify it in the network |
| **Internet** | The global service that connects many servers and networks across the WAN |

**Why "logical"?** The MAC address is burned into the NIC (physical, permanent). The IP address is *assigned* by software and can change, like a mobile number versus your Aadhaar number.

### Port numbers

In networking, **every service has a port number**. The port number tells the computer **which service** the data is meant for.

**Simple example:** IP address = **building address**. Port number = **flat number** inside the building.

| Port | Service |
|---|---|
| 22 | SSH (remote login to Linux) |
| 80 | HTTP (websites) |
| 443 | HTTPS (secure websites) |
| 3389 | RDP (remote login to Windows) |
| 3306 | MySQL database |

You already used ports 22 and 80 in the Day 2 lab (security group rules).

> ⚠️ **Correction to class notes:** The notes say *"IP is the basic communication service that starts with port number 0"*. IP itself **does not use port numbers**. Ports belong to the protocols that run *on top of* IP: **TCP** and **UDP**. Port numbers range from **0 to 65535**, and port **0 is reserved** (not used for a real service). IP identifies the **device**; TCP/UDP ports identify the **service** on that device.

---

## 5. Level 3 – Architect view

### OSI and TCP/IP layers (where each piece lives)

| OSI layer | Name | Example | Device / AWS link |
|---|---|---|---|
| 7 | Application | HTTP, DNS, SSH | Application Load Balancer, Route 53 |
| 4 | Transport | **TCP, UDP (ports)** | Network Load Balancer, security group port rules |
| 3 | Network | **IP (addresses)** | Router, VPC route tables |
| 2 | Data link | Ethernet (**MAC**) | Switch |
| 1 | Physical | Cables, Wi-Fi signals | NIC, hub |

### Mapping traditional devices to AWS

| Traditional | AWS equivalent |
|---|---|
| Your own office network (LAN) | **VPC** (Virtual Private Cloud) |
| Smaller network segments | **Subnets** |
| Router | **Route tables** (+ implicit VPC router) |
| Connection to ISP / internet | **Internet Gateway** |
| Firewall at the server | **Security Group** (stateful) |
| Firewall at the network segment | **Network ACL** (stateless) |
| NIC | **ENI** (Elastic Network Interface) |
| Leased line between offices | **Direct Connect** / **Site-to-Site VPN** |

> 🎯 **Exam tip:** Security groups work on **ports and IP ranges**. If a question says "users can't reach the website", check that the security group allows **port 80/443** inbound.

---

## 6. Key terms

| Term | Meaning |
|---|---|
| Network | Computers connected by wired or wireless media |
| Networking | Computers communicating over a network |
| LAN / MAN / WAN | Local / city-wide / global network |
| NIC | Card that connects a device to a network (has a MAC address) |
| Switch | Connects devices inside a LAN, delivers to the right device |
| Router | Connects different networks |
| Firewall | Allows or blocks traffic by rules |
| Protocol | Set of rules for communication |
| IP address | Logical address of a device (IPv4 32-bit, IPv6 128-bit) |
| Port number | Identifies a service on a device (0–65535) |

---

## 7. Quick quiz

1. What is the difference between a network and networking?
2. A bank connects its branches across one city. Which network type is this?
3. Which device delivers data only to the right PC inside a LAN?
4. Which device connects your office network to the internet?
5. How many bits are in an IPv4 address? In IPv6?
6. A website uses which port? A secure website?
7. Does IP use port numbers?

<details>
<summary>Show answers</summary>

1. Network = the connection; networking = the communication over it
2. MAN
3. Switch
4. Router (usually with a firewall in front)
5. 32 bits; 128 bits
6. 80 (HTTP); 443 (HTTPS)
7. No. Ports belong to TCP and UDP, which run on top of IP

</details>

---

## One-line summary

> **A network connects computers using devices like NICs, switches and routers; IP addresses identify each device, and port numbers identify each service on it.**
