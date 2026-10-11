# Day 6 – IPv4 Addressing

**Today's agenda**
- IPv4 structure (32 bits)
- Binary to decimal conversion
- Range of IP addresses and special addresses
- Valid and invalid IP addressing between PCs
- Subnet mask and why IP addresses were classified

---

## 1. Level 1 – Basics: What does an IPv4 address look like?

> **IPv4 = 32 bits**, written as **four numbers separated by dots**.

```
   IP:   X . X . X . X          e.g.  192.168.1.10
        [8] [8] [8] [8]   bits  = 32 bits in total
```

- A **bit** is a binary number: **0 or 1**.
- Each part (called an **octet**) has **8 bits**.
- 4 octets × 8 bits = **32 bits**.

### What values can each X take?

| | Binary (8 bits) | Decimal |
|---|---|---|
| **Least** | `0 0 0 0 0 0 0 0` | **0** |
| **Highest** | `1 1 1 1 1 1 1 1` | **255** |

So each **X = 0 to 255** (256 possible values).

---

## 2. Binary → Decimal (the conversion trick)

### Powers of 2 – remember this table

| Position | 2⁷ | 2⁶ | 2⁵ | 2⁴ | 2³ | 2² | 2¹ | 2⁰ |
|---|---|---|---|---|---|---|---|---|
| **Value** | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |

**Memory trick:** start at 1 and keep doubling: 1, 2, 4, 8, 16, 32, 64, 128. Add all eight and you get **255**.

### Worked example (from class): `10110001`

```
 Bits:     1    0    1    1    0    0    0    1
 Value:  128   64   32   16    8    4    2    1
          ↓          ↓    ↓                   ↓
         128 +  0 + 32 + 16 +  0 +  0 +  0 +  1  = 177
```

= 1×2⁷ + 0×2⁶ + 1×2⁵ + 1×2⁴ + 0×2³ + 0×2² + 0×2¹ + 1×2⁰
= 128 + 0 + 32 + 16 + 0 + 0 + 0 + 1 = **177**

**Rule:** write the place values under the bits, then add the values wherever the bit is **1**.

### Decimal → Binary (reverse)

Go left to right through 128, 64, 32 … If the value fits in what's left, write **1** and subtract; otherwise write **0**.

Example: **192**
- 128 fits → 1 (left 64) → 64 fits → 1 (left 0) → rest are 0
- **192 = 11000000**

### Practice

| Decimal | Binary |
|---|---|
| 255 | 11111111 |
| 192 | 11000000 |
| 168 | 10101000 |
| 10 | 00001010 |
| 1 | 00000001 |

So `192.168.1.10` in binary = `11000000.10101000.00000001.00001010`

---

## 3. Special addresses (from class)

| Address | Name | Meaning |
|---|---|---|
| **0.0.0.0** | Null / "any" address | Means "this device" or "**all addresses**". Not given to a PC |
| **255.255.255.255** | Universal (limited) broadcast | The last IPv4 address; a message to **every device** on the local network |
| **1.x.x.x** | e.g. `1.0.0.1` | A normal address where the first octet is 1 |

> 🔗 **AWS link:** You'll see `0.0.0.0/0` everywhere in AWS. In a security group or route table it means **"anywhere on the internet"** (all IPv4 addresses).

> ⚠️ **Clarification:** The notes say the valid range for the first octet is **1–255**. In practice, only **1–223** are used for normal devices:
> - **127.x.x.x** = loopback (the computer talking to itself, e.g. `127.0.0.1` = localhost)
> - **224–239** = multicast (one-to-many groups)
> - **240–255** = reserved for research
>
> You'll see why when we cover IP classes.

---

## 4. Level 2 – Can these two PCs talk? (Valid vs invalid addressing)

Two PCs connected directly with a cable:

```
   PC1 ────────────── PC2
```

| PC1 | PC2 | Result | Why |
|---|---|---|---|
| 1.0.0.1 | 1.0.0.0 | ❌ | `1.0.0.0` is the **Network ID**, the starting point of the network. It names the network, so it can't be given to a device |
| 1.0.0.1 | 1.0.0.1 | ❌ | Same IP on two devices = **IP conflict** |
| 1.0.0.1 | 1.255.255.255 | ❌ | `1.255.255.255` is the **last address** of network "1", called the **Broadcast address**. Reserved for messages to all devices |
| 1.0.0.1 | 2.0.0.1 | ❌ | **Different networks.** They need a **router** to talk |
| 1.0.0.1 | 1.0.0.2 | ✅ | Same network, unique addresses: **valid addressing** |

### Two reserved addresses in every network

```
 Network 1 (subnet mask 255.0.0.0)

 1.0.0.0  ← Network ID (first)     ❌ not for devices
 1.0.0.1  ┐
   ...    ├ usable host addresses  ✅
 1.255.255.254 ┘
 1.255.255.255 ← Broadcast (last)  ❌ not for devices
```

> 💡 **Fine print:** "The last block should not be 0" is a good rule for small networks, but the real rule is about the **whole host part**. In network 1 (mask 255.0.0.0), `1.0.5.0` is a perfectly valid device address. Only `1.0.0.0`, where *all* the host bits are 0, is the network ID.

---

## 5. Subnet mask – which part is constant and which is variable?

Take the network **1.0.0.0**. If the first number **1** stays constant, how many addresses do we get?

```
   1  .  (0–255)  .  (0–255)  .  (0–255)
 fixed     256         256         256

 Total = 256 × 256 × 256 = 1,67,77,216  (16,777,216 addresses)
 Range: 1.0.0.0 → 1.255.255.255
```

An IP address has **constant** parts (the **network**) and **variable** parts (the **hosts**/devices). To show which is which, the **subnet mask** was introduced.

| Rule | Value in mask |
|---|---|
| **Constant** (network part) | Highest value → **255** |
| **Variable** (host part) | Least value → **0** |

```
 IP:           1 .   X .   X .   X
 Subnet mask: 255 .  0 .   0 .   0      → "first octet is the network, rest are devices"
```

### Going deeper: the mask in binary and CIDR

```
 255.0.0.0  = 11111111.00000000.00000000.00000000
              └─ 8 ones = network bits ─┘
 Short form (CIDR): 1.0.0.0/8
```

| Subnet mask | CIDR | Network part | Addresses |
|---|---|---|---|
| 255.0.0.0 | /8 | 1st octet | 16,777,216 |
| 255.255.0.0 | /16 | first 2 octets | 65,536 |
| 255.255.255.0 | /24 | first 3 octets | 256 |

**Formula:** addresses = 2^(host bits). Usable for devices = 2^(host bits) − 2 (minus network ID and broadcast).

---

## 6. Why IP addresses were classified

If a whole network like `1.x.x.x` (16.7 million addresses) were sold to one company:

1. **Most companies don't have that many devices.**
2. If each company got an entire first-octet network, there would be networks for **only about 255 companies** in the world.
3. A company buying such a range would **waste almost all** of the addresses.

**Solution:** **IANA** (Internet Assigned Numbers Authority), with standards from the **IETF** (Internet Engineering Task Force), **classified** IP addresses into classes of different sizes (Class A, B, C, D, E), so organizations get a block that fits their needs.

➡️ **Coming next:** IP classes, private vs public IPs, and subnetting.

---

## 7. Level 3 – Architect view

- **Classes didn't solve the waste fully.** In 1993 the internet moved to **CIDR** (Classless Inter-Domain Routing): any prefix length (`/8` to `/32`) can be used, not just /8, /16, /24. AWS uses **CIDR notation everywhere**.
- **Private IP ranges (RFC 1918)**, not routable on the internet, used inside companies and VPCs:
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16` (the `192.168.1.1` from Day 5 is a private address)
- **AWS VPC sizing:** a VPC CIDR can be from **/16** (65,536 addresses) to **/28** (16 addresses).
- **AWS reserves 5 IPs in every subnet**, not just 2: network address, **.1** (VPC router), **.2** (DNS), **.3** (future use) and the broadcast address. So a `/24` subnet gives **251** usable IPs, not 254.
- **Plan for no overlap:** if your VPC CIDR overlaps with your office network or another VPC, you **can't connect them** later (VPN, peering, Transit Gateway). Choose ranges carefully on day one.
- **IPv4 exhaustion** is why **IPv6** (128-bit, written in hex like `2001:db8::1`) exists. AWS supports dual-stack VPCs, and since February 2024 AWS charges for every public IPv4 address, a strong push toward IPv6.

> 🎯 **Exam tip:** "How many usable IPs in a /24 subnet in AWS?" → **251** (256 − 5).

---

## 8. Key terms

| Term | Meaning |
|---|---|
| Bit | A binary digit: 0 or 1 |
| Octet | 8 bits; one of the four parts of an IPv4 address (0–255) |
| IPv4 | 32-bit address, four octets |
| Network ID | First address of a network (all host bits 0) – not for devices |
| Broadcast address | Last address of a network (all host bits 1) – message to all |
| IP conflict | Two devices with the same IP |
| Subnet mask | Shows the network part (255) and host part (0) |
| CIDR | Short form of the mask, e.g. /8, /16, /24 |
| 0.0.0.0/0 | "All IPv4 addresses" – anywhere |
| IANA | Organization that manages global IP address allocation |
| IETF | Group that writes internet standards |

---

## 9. Quick quiz

1. How many bits are in an IPv4 address, and in each octet?
2. Convert `11000000` to decimal.
3. Convert `10101000` to decimal.
4. What is the range of values for one octet?
5. Can PC1 = `1.0.0.1` and PC2 = `1.0.0.0` communicate? Why?
6. Can PC1 = `1.0.0.1` and PC2 = `2.0.0.1` communicate directly?
7. In network 1 with mask 255.0.0.0, what is the broadcast address?
8. What does a subnet mask of 255 mean? What does 0 mean?
9. How many addresses are in `1.0.0.0/8`?
10. *(AWS)* How many usable IPs does a /24 subnet give you in AWS?

<details>
<summary>Show answers</summary>

1. 32 bits; 8 bits per octet
2. 192 (128 + 64)
3. 168 (128 + 32 + 8)
4. 0 to 255
5. No. 1.0.0.0 is the network ID and can't be assigned to a device
6. No. They are in different networks and need a router
7. 1.255.255.255
8. 255 = constant (network part); 0 = variable (host part)
9. 256 × 256 × 256 = 16,777,216
10. 251 (AWS reserves 5)

</details>

---

## One-line summary

> **An IPv4 address is 32 bits in four octets (0–255); the subnet mask splits it into a fixed network part and a variable host part, and the first (network ID) and last (broadcast) addresses are never given to devices.**
