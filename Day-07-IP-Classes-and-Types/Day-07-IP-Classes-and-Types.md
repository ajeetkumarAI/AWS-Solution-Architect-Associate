# Day 7 – IP Addressing (Part 2): Classes, Types & Assignment

**Today's agenda**
- Classes of IP address
- Types of IP address (public and private)
- Special IP addresses
- How to assign an IP address (static vs dynamic)

> **Quick recap from Day 6:** An IPv4 address has 32 bits in 4 octets (0–255). The **network** part is **constant** and the **host** part is **variable**. The subnet mask marks them: **255 = network**, **0 = host**. Giving one company a whole `1.x.x.x` network (16.7 million addresses) wastes almost all of them, so IANA and the IETF **classified** IP addresses.

---

## 1. Level 1 – Basics: The idea behind classes

```
                 IP Address
                /          \
          Constants      Variables
              │              │
          Networks         Hosts
              │              │
        Highest value    Least value
     (binary 1s → 255)  (binary 0s → 0)
```

**Simple example:** Think of an address like `Building . Flat`.
- A **big** organization needs few buildings but a huge number of flats → **Class A**
- A **medium** organization → **Class B**
- A **small** office or home needs many small buildings with few flats → **Class C**

The **first octet** of an IP address tells you its class.

| Class | First octet | Shape | Default subnet mask | Used for |
|---|---|---|---|---|
| **A** | **1 – 126** | N . H . H . H | 255.0.0.0 | Large organizations, government |
| **B** | **128 – 191** | N . N . H . H | 255.255.0.0 | MNCs |
| **C** | **192 – 223** | N . N . N . H | 255.255.255.0 | Small companies, homes |
| **D** | **224 – 239** | No N/H split | No subnet mask | Multicasting (e.g. `224.0.0.1`) |
| **E** | **240 – 255** | No N/H split | No subnet mask | Research & development |

`N` = network octet (constant), `H` = host octet (variable).

**Where did 127 go?** It's missing between A and B on purpose. `127.x.x.x` is reserved for **loopback** (see section 4).

**Memory trick:** the classes start at **1 – 128 – 192 – 224 – 240**. Each start is the previous one plus half of what is left up to 256 (128 → +64 = 192 → +32 = 224 → +16 = 240).

---

## 2. Level 2 – Going deeper: Counting networks and hosts

**Two formulas**
- Addresses per network = 256 for every H octet → 256 × 256 × …
- **Valid (usable) hosts = total − 2**, because the first address is the **Network ID** and the last is the **Broadcast** address.

### Class A – `N.H.H.H`

- Networks: first octet 1–126 → **126 networks**
- Hosts per network: 3 host octets = 256 × 256 × 256 = 1,67,77,216
- **Valid hosts = 1,67,77,214** (16,777,214)
- Example: network `10.0.0.0`, hosts `10.0.0.1` → `10.255.255.254`

### Class B – `N.N.H.H`

- Networks: first octet has 64 values (128–191) × second octet 256 values = **64 × 256 = 16,384 networks**
- Hosts per network: (256 × 256) − 2 = **65,534**
- Example: network `172.16.0.0`, hosts `172.16.0.1` → `172.16.255.254`

**Class B check (from class):**

```
   PC1 128.0.0.1  ────✗────  PC2 128.1.0.1
        └─┬─┘                     └─┬─┘
       network 128.0             network 128.1   → different networks, can't talk directly
```

In Class B the **first two** octets are the network, so `128.0` and `128.1` are **different networks**.

### Class C – `N.N.N.H`

- Networks: first octet has 32 values (192–223) × 256 × 256 = **20,97,152 networks** (2,097,152)
- Hosts per network: 256 − 2 = **254**
- Example: network `192.168.1.0`, hosts `192.168.1.1` → `192.168.1.254`, broadcast `192.168.1.255`

### Summary table

| Class | Networks | Valid hosts per network | Mask | CIDR |
|---|---|---|---|---|
| A | 126 | 1,67,77,214 (16,777,214) | 255.0.0.0 | /8 |
| B | 16,384 | 65,534 | 255.255.0.0 | /16 |
| C | 20,97,152 (2,097,152) | 254 | 255.255.255.0 | /24 |

### How a computer sees the class (the leading bits)

The class is fixed by the **first few bits** of the first octet:

| Class | First bits | First octet in binary | Range |
|---|---|---|---|
| A | `0` | `0xxxxxxx` | 0 – 127 |
| B | `10` | `10xxxxxx` | 128 – 191 |
| C | `110` | `110xxxxx` | 192 – 223 |
| D | `1110` | `1110xxxx` | 224 – 239 |
| E | `1111` | `1111xxxx` | 240 – 255 |

This is why Class B has **64** first-octet values (6 free bits = 2⁶) and Class C has **32** (5 free bits = 2⁵).

---

## 3. Types of IP address: Public vs Private

| | Public IP | Private IP |
|---|---|---|
| Works on | The **internet** | Only **inside** an organization / home network |
| Unique? | Unique across the whole world | Can repeat in different organizations |
| Cost | **Paid** (you get it from an ISP or cloud provider) | **Free** |
| Who gives it | ISP / IANA (through regional registries) | Your own admin or router |
| Example | `8.8.8.8` (Google DNS) | `192.168.1.10` (your home laptop) |

**Public range:** first octet **1–223**, excluding **127** and the private and special ranges below.

### Private ranges (set aside by IETF and IANA)

| Class | Private range | CIDR |
|---|---|---|
| A | `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` |
| B | `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` |
| C | `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

**Memory trick:** **10, 172.16–31, 192.168**.

**How do private devices reach the internet?** Your home router uses **NAT (Network Address Translation)**: it swaps your private IP for the router's single public IP on the way out, and swaps it back on the way in. That's how 10 devices at home share one public IP.

> ⚠️ **Correction to class notes:** The notes say public IPs "differ in bandwidth/speed". The IP address itself has **no speed**. Speed (bandwidth) comes from your **ISP plan**. What you pay for with a public IP is a **globally unique, internet-reachable address**.

---

## 4. Special IP addresses

| Address | Range | What it means |
|---|---|---|
| **Loopback** | `127.0.0.0` – `127.255.255.255` (usually `127.0.0.1`, "localhost") | The computer sends data **to itself**. Used to test that the network software is working |
| **Link-local / APIPA** (Automatic Private IP Addressing) | `169.254.0.0` – `169.254.255.255` | When a device **can't find a DHCP server**, it gives itself one of these. Seeing `169.254.x.x` on your PC is a sign that something is wrong |
| **Network ID** | First address of a network | Names the network, not given to a device |
| **Broadcast** | Last address of a network | Message to all devices in that network |
| **0.0.0.0** | — | "This device / any address" |
| **255.255.255.255** | — | Broadcast to every device on the local network |

> 💡 **Clarification:** The notes say loopback "verifies the working status of the NIC". Loopback traffic never actually leaves the computer or touches the network card. A successful `ping 127.0.0.1` proves the computer's **TCP/IP software** works. To test the NIC, ping your **own real IP** (e.g. `ping 192.168.1.10`).

**Try it yourself (Windows Command Prompt):**

```
ipconfig          → shows your IP, subnet mask, default gateway
ipconfig /all     → also shows if DHCP is enabled and which DHCP server gave the IP
ping 127.0.0.1    → tests your own TCP/IP software (loopback)
ipconfig /release
ipconfig /renew   → gives the DHCP IP back and asks for a new one
```

---

## 5. Assigning an IP address

| Method | How | Good for |
|---|---|---|
| **Static** | **Manually** typed in by a user or admin. Doesn't change | Servers, printers, routers – anything others must always find at the same address |
| **Dynamic** | **Automatically** given by a **DHCP server** | Laptops, phones – devices that come and go |

> **DHCP – Dynamic Host Configuration Protocol:** automatically assigns IP addresses (plus subnet mask, gateway and DNS server) to devices in the network.

### How DHCP works – "DORA"

```
 Device                                  DHCP Server
   │ ── D: Discover  "Any DHCP server here?" ──▶ │
   │ ◀── O: Offer    "You can use 192.168.1.20" ─ │
   │ ── R: Request   "Yes, I'll take it" ───────▶ │
   │ ◀── A: Acknowledge "It's yours for 24 hrs" ─ │
```

The IP is **leased** for a time and renewed automatically. If no server answers the Discover, the device falls back to **APIPA (169.254.x.x)**.

---

## 6. Level 3 – Architect view

- **Classes are history; CIDR is reality.** Classful addressing was replaced by **CIDR** in 1993 because fixed sizes (/8, /16, /24) still wasted addresses. AWS **never** asks for a class. It asks for a **CIDR block** like `10.0.0.0/16`. You still need classes to understand where the private ranges and default masks come from.
- **AWS VPCs use the private ranges.** Most VPCs are carved from `10.0.0.0/8`, `172.16.0.0/12` or `192.168.0.0/16`. (Allowed VPC size: /16 to /28.) Avoid `172.17.0.0/16`, which some AWS services and Docker use by default.
- **Public IPs in AWS:**
  - **Auto-assigned public IP** – changes every time you stop and start an EC2 instance.
  - **Elastic IP** – a **static** public IP you own until you release it.
  - Since February 2024, AWS charges for **every** public IPv4 address, in use or not.
- **NAT in AWS:** EC2 instances in a **private subnet** reach the internet through a **NAT Gateway** (just like your home router), while an **Internet Gateway** gives public subnets two-way internet access.
- **DHCP in AWS:** every EC2 instance gets its **private IP** through the VPC's built-in DHCP service. You can customize DNS settings with **DHCP option sets**.
- **Link-local in AWS:** `169.254.169.254` is the **Instance Metadata Service (IMDS)**. From inside an instance it returns the instance ID, IAM role credentials and more. Always enforce **IMDSv2** (token-based) to protect against credential theft.

> 🎯 **Exam tips:**
> - "Instance needs a public IP that doesn't change after reboot/stop" → **Elastic IP**.
> - "Instances in a private subnet need to download updates but must not be reachable from the internet" → **NAT Gateway**.
> - "Application needs its own IAM credentials from inside EC2" → instance **role** via **metadata (169.254.169.254)**, never hard-coded keys.

---

## 7. Key terms

| Term | Meaning |
|---|---|
| Class A / B / C | Network sizes for large / medium / small organizations |
| Class D | Multicast (224–239) |
| Class E | Research (240–255) |
| Public IP | Internet-reachable, globally unique, paid |
| Private IP | Used inside a network only, free, can repeat across organizations |
| NAT | Translates private IPs to a public IP to reach the internet |
| Loopback | 127.x.x.x – device talking to itself |
| APIPA / link-local | 169.254.x.x – self-assigned when DHCP fails |
| Static IP | Set manually, doesn't change |
| Dynamic IP | Given automatically by DHCP |
| DHCP | Protocol that assigns IPs automatically (DORA) |
| Elastic IP | AWS static public IPv4 address |

---

## 8. Quick quiz

1. Which class does `150.10.5.4` belong to? What is its default subnet mask?
2. Which class does `200.1.1.1` belong to? How many valid hosts per network?
3. Why is `127` missing from Class A's range?
4. Can `128.0.0.1` and `128.1.0.1` talk directly? Why?
5. Is `172.20.5.9` public or private? What about `172.40.5.9`?
6. Your PC shows `169.254.33.7`. What does it mean?
7. What is the difference between static and dynamic IP assignment?
8. What do the four letters of DORA stand for?
9. How many networks are in Class B, and how was that calculated?
10. *(AWS)* Your EC2 instance's public IP changed after a stop/start. What should you use instead?

<details>
<summary>Show answers</summary>

1. Class B; 255.255.0.0
2. Class C; 254
3. 127.x.x.x is reserved for loopback
4. No. In Class B the first two octets are the network, so 128.0 and 128.1 are different networks
5. `172.20.5.9` is private (172.16–172.31); `172.40.5.9` is public
6. The device couldn't reach a DHCP server and gave itself an APIPA address
7. Static = set manually by an admin; dynamic = given automatically by a DHCP server
8. Discover, Offer, Request, Acknowledge
9. 16,384 = 64 (first octet 128–191) × 256 (second octet)
10. An Elastic IP

</details>

---

## One-line summary

> **IPv4 addresses are grouped into classes A, B, C (for devices), D (multicast) and E (research); private ranges (10, 172.16–31, 192.168) are free for internal use, public IPs reach the internet, and DHCP hands out addresses automatically.**
