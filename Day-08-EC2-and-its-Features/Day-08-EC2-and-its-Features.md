# Day 8 – Amazon EC2 and its Features

**Today's agenda**
- What is Amazon EC2?
- Features of EC2
- Traditional IT infrastructure vs AWS (cloud) infrastructure

> **Where we are:** Days 5–7 covered networking and IP addresses. Today we use that knowledge in AWS's most important service: **EC2**, the virtual server.

---

## 1. Level 1 – Basics: What is Amazon EC2?

**EC2 = Elastic Compute Cloud**

| Word | Meaning |
|---|---|
| **Elastic** | Stretches and shrinks: add servers when busy, remove them when quiet |
| **Compute** | Processing power (CPU, RAM) to run your applications |
| **Cloud** | Delivered over the internet, pay-as-you-go |

> **Definition (AWS):** Amazon EC2 provides **on-demand, scalable computing capacity** in the AWS Cloud. It reduces hardware costs so you can develop and deploy applications faster.
>
> **An EC2 instance is a virtual server in the AWS Cloud.**

**Simple example:** Renting a car instead of buying one. You choose the model (small hatchback or big SUV), use it for as long as you need, return it, and pay only for the time you used it. EC2 is the same for servers.

### What EC2 lets you do

- Launch **as many or as few** virtual servers as you need
- Configure **security** and **networking**
- Manage **storage**
- **Scale up** for compute-heavy tasks (monthly or yearly processing, traffic spikes during a sale)
- **Scale down** when usage drops, so you stop paying for idle capacity

> 🔗 **Connect the dots:** On Day 4 you learned virtualization: one physical server running many VMs. An EC2 instance **is** one of those VMs, running on AWS's **Nitro hypervisor** inside an Availability Zone. On Day 2 you launched one in the web-server lab.

---

## 2. Traditional IT vs AWS Infrastructure – the mapping (from class)

Everything you used to buy and manage yourself in a traditional setup has an EC2 feature that replaces it:

| # | Traditional IT | → | AWS / EC2 feature | What it does |
|---|---|---|---|---|
| 1 | **Physical servers** | → | **Instances** | Virtual servers |
| 2 | **OS & software** | → | **AMIs** (Amazon Machine Images) | Ready template with the OS and software pre-installed |
| 3 | **Server hardware** (CPU, RAM, disk, network card) | → | **Instance types** | Choose the hardware size: CPU, memory, storage, network, GPU |
| 4 | **Storage devices** (hard disks, SAN) | → | **EBS volumes** (Elastic Block Store) | Permanent disks for your instance |
| 5 | Disk built into the server (local scratch disk) | → | **Instance store volumes** | Temporary disk, data is lost when the instance stops |
| 6 | **Credentials** (username/password) | → | **Key pairs** | Secure login: AWS keeps the public key, you keep the private key |
| 7 | **Firewall** (security) | → | **Security groups** | Virtual firewall: which ports and IPs can reach the instance |
| 8 | **Local network (LAN)** | → | **VPC** (Virtual Private Cloud) | Your own private, isolated network in AWS |
| 9 | **Server room** | → | **Regions & Availability Zones** | Where in the world your servers physically run |
| 10 | **Labels** (stickers on servers) | → | **Tags** | Key–value labels like `Name=WebServer`, `Env=Prod` |
| 11 | **ISP (public IP)** | → | **Elastic IP** | A fixed public IPv4 address you own |

> 💡 **Note:** In the class diagram, **Instance store volumes** had no line from the traditional side. Its closest match is the **disk built inside a physical server** used for temporary/scratch data. That's what I've added as row 5.

### Picture it

```
TRADITIONAL OFFICE                         AWS CLOUD
──────────────────                         ─────────
Server room ─────────────────────────────▶ Region / Availability Zone
 └─ LAN (switch, cables) ────────────────▶  └─ VPC (+ subnets)
     └─ Firewall ────────────────────────▶      └─ Security group
         └─ Physical server ─────────────▶          └─ EC2 instance
             ├─ Hardware (CPU/RAM) ──────▶              ├─ Instance type (t3.micro…)
             ├─ OS + software ───────────▶              ├─ AMI
             ├─ Hard disks ──────────────▶              ├─ EBS volume / Instance store
             ├─ Login password ──────────▶              ├─ Key pair
             ├─ Sticker label ───────────▶              ├─ Tags
             └─ ISP public IP ───────────▶              └─ Elastic IP
```

---

## 3. Level 2 – Going deeper: Each feature in detail

### 3.1 Instances – virtual servers

An **instance** is the running virtual server. It moves through a **lifecycle**:

```
 Launch → pending → running ⇄ stopping → stopped → (start again) → running
                       │
                    rebooting
                       │
                 shutting-down → terminated (gone forever)
```

| Action | What happens | Billing |
|---|---|---|
| **Stop** | Like shutting down a PC. EBS data stays | Instance charge stops; **EBS storage still billed** |
| **Reboot** | Like restarting. Same host, same IPs | Still running, still billed |
| **Terminate** | Deleted permanently (root EBS deleted by default) | Billing stops |

Linux instances are billed **per second** (minimum 60 seconds).

### 3.2 AMIs – Amazon Machine Images

An **AMI** is a **pre-configured template** for an instance: the operating system plus any extra software you need.

**Simple example:** A **Windows installation CD with Office already installed**. Every PC you set up from it is identical.

| Source | Examples |
|---|---|
| **AWS-provided** | Amazon Linux 2023, Ubuntu, Red Hat, Windows Server |
| **AWS Marketplace** | Ready-made images from vendors (e.g. WordPress, firewalls), sometimes paid |
| **Community AMIs** | Shared publicly by other users (check carefully before use) |
| **My AMIs** | Your own: configure one instance, then create an AMI from it to launch identical copies |

AMIs are **Region-specific**. You can **copy** an AMI to another Region.

### 3.3 Instance types – the hardware size

The instance type decides the **CPU, memory, storage, network and GPU** of your instance.

**How to read the name** – e.g. `t3.micro`, `m7g.large`:

```
   m   7   g   .  large
   │   │   │       └── Size: nano, micro, small, medium, large, xlarge, 2xlarge…
   │   │   └────────── Extra attribute: g = AWS Graviton (ARM) CPU, i = Intel, a = AMD
   │   └────────────── Generation (higher = newer, usually cheaper per performance)
   └────────────────── Family (what it's optimized for)
```

| Family | Letters | Optimized for | Example use |
|---|---|---|---|
| **General purpose** | **T**, **M** | Balanced CPU and memory | Web servers, small apps, dev/test |
| **Compute optimized** | **C** | High CPU | Batch processing, gaming servers, video encoding |
| **Memory optimized** | **R**, **X** | Lots of RAM | Big databases, in-memory caches |
| **Storage optimized** | **I**, **D** | Fast local disk | Data warehousing, NoSQL databases |
| **Accelerated computing** | **P**, **G**, **Inf**, **Trn** | GPUs / AI chips | Machine learning, graphics |

**Memory trick:** **C**ompute = **C**PU, **R** = **R**AM, **I** = **I**/O (disk), **G** = **G**raphics, **T** = **T**iny/burstable.

> **T family = burstable.** It runs at a low baseline and earns **CPU credits** while idle, then spends them to "burst" when busy. Great for websites with light, uneven traffic.

### 3.4 EBS volumes – persistent storage

**EBS (Elastic Block Store)** = a **network hard disk** attached to your instance.

- **Persistent** – data survives stop/start (and termination too, if you untick "Delete on termination").
- Lives in **one Availability Zone**, and can attach only to instances in that same AZ.
- Can be **resized** without downtime.
- Back it up with **snapshots** (stored in S3, can be copied to other Regions).

| EBS type | Kind | Use |
|---|---|---|
| **gp3** (General Purpose SSD) | SSD | Default for most workloads, boot volumes |
| **io2** (Provisioned IOPS SSD) | SSD | Critical databases needing very high, steady speed |
| **st1** (Throughput Optimized HDD) | HDD | Big sequential data: logs, data processing |
| **sc1** (Cold HDD) | HDD | Rarely accessed data, lowest cost |

**Simple example:** An **external hard disk** plugged into your laptop. Switch the laptop off and the files are still on the disk.

### 3.5 Instance store volumes – temporary storage

- Disk **physically attached** to the host computer → **very fast**.
- **Temporary:** data is **deleted** when you **stop, hibernate or terminate** the instance (it survives a reboot).
- Included in the instance price, but only certain instance types have it.

**Simple example:** A **whiteboard** in a rented meeting room. Very handy while you're in the room, wiped clean when you leave.

| | EBS volume | Instance store |
|---|---|---|
| Connection | Over the network | Physically attached |
| Speed | Fast | Very fast |
| Data after stop | ✅ Kept | ❌ Lost |
| Detach and move | ✅ Yes (same AZ) | ❌ No |
| Snapshots | ✅ Yes | ❌ No |
| Use for | OS, databases, anything important | Cache, buffers, scratch/temp data |

### 3.6 Key pairs – secure login

A key pair = **public key + private key**.

- **AWS stores the public key** (placed inside the instance).
- **You store the private key** (`.pem` file) in a safe place. **AWS doesn't keep a copy.** Lose it and you can't log in with it.

```
  Your laptop                          EC2 instance
  ┌───────────────┐   SSH (port 22)    ┌──────────────────┐
  │ private key   │ ─────────────────▶ │ public key       │
  │ my-key.pem    │                    │ (matches → login)│
  └───────────────┘                    └──────────────────┘
```

**Simple example:** A **lock and key**. AWS installs the **lock** (public key) on the door. Only **your key** (private key) opens it.

**Login commands**

```bash
# Linux / macOS / Windows PowerShell
chmod 400 my-key.pem                       # Linux/macOS only: protect the key file
ssh -i my-key.pem ec2-user@<public-ip>     # Amazon Linux user = ec2-user
ssh -i my-key.pem ubuntu@<public-ip>       # Ubuntu user = ubuntu
```

- **Windows instances:** the private key is used to **decrypt the Administrator password**, then you log in with **RDP (port 3389)**.
- `.pem` → for OpenSSH. `.ppk` → for PuTTY.
- No key needed when using **EC2 Instance Connect** or **Session Manager** from the console.

### 3.7 Security groups – virtual firewall

A **security group** controls **which traffic can reach** your instance (inbound) and **where it can connect** (outbound), using **protocol + port + source/destination IP**.

| Rule | Type | Protocol | Port | Source |
|---|---|---|---|---|
| Inbound | SSH | TCP | 22 | My IP (e.g. `49.36.10.5/32`) |
| Inbound | HTTP | TCP | 80 | `0.0.0.0/0` (anywhere) |
| Inbound | HTTPS | TCP | 443 | `0.0.0.0/0` |
| Outbound | All traffic | All | All | `0.0.0.0/0` |

**Key rules to remember**
- **Allow rules only** – you can't write a "deny" rule.
- By default: **all inbound blocked**, **all outbound allowed**.
- **Stateful** – if a request is allowed in, the reply is automatically allowed out.
- One instance can have **several** security groups; one security group can be used by **many** instances.
- Changes apply **immediately**.

> 🔗 **Connect the dots:** You used Day 5 (**ports**) and Days 6–7 (**IP addresses, /32, 0.0.0.0/0**) right here. `/32` means exactly one IP address, and `0.0.0.0/0` means the whole internet.

### 3.8 VPC, Regions/AZs, Tags, Elastic IP (quick view)

| Feature | Key points |
|---|---|
| **VPC** | Your private network in AWS. Every instance launches into a VPC **subnet** and gets a **private IP** from it (Day 7: private ranges like `10.0.0.0/16`). Each account has a **default VPC** in each Region |
| **Regions & AZs** | Choose the Region (e.g. Mumbai `ap-south-1`) and AZ when launching. Spread instances across **multiple AZs** for high availability |
| **Tags** | Key–value labels (`Name=web-01`, `Env=prod`, `Owner=Ajeetkumar`). The **Name** tag is what shows in the console. Used to search, group, **track costs** and control access |
| **Elastic IP** | A **static public IPv4** address. The normal public IP **changes** every stop/start; an Elastic IP **doesn't**. Charged per hour whether attached or not, so **release** it when you don't need it |

---

## 4. Traditional infrastructure vs Cloud infrastructure (full comparison)

| Point | Traditional (on-premises) | Cloud (AWS EC2) |
|---|---|---|
| **Getting a server** | Order, ship, rack, cable, install: **weeks** | Click **Launch**: **minutes** |
| **Cost model** | Big upfront purchase (**CapEx**), e.g. ₹7 lakh per server | **Pay-as-you-go** (**OpEx**), per second |
| **Capacity planning** | Guess years ahead; usually over-buy (Day 4: only ~17% used) | Match real demand; scale up and down |
| **Scaling** | Buy more hardware, wait for delivery | Change instance type or add instances; **Auto Scaling** does it automatically |
| **OS setup** | Install OS and software on each server by hand | Launch from an **AMI**, identical every time |
| **Storage** | Buy disks/SAN; replacing is physical work | **EBS** volumes created, resized and snapshotted in clicks |
| **Security (firewall)** | Hardware firewall appliance | **Security groups** (+ network ACLs), free |
| **Login / credentials** | Passwords managed per server | **Key pairs**, IAM roles, Session Manager |
| **Network** | Switches, routers, cables in a LAN | **VPC**, subnets, route tables: software-defined |
| **Public IP** | Bought from the ISP | **Elastic IP** in seconds |
| **Location** | One server room in one city | **39 Regions, 124 AZs** worldwide |
| **High availability / DR** | Build a second data center (very expensive) | Deploy in multiple AZs/Regions |
| **Maintenance** | Your team: power, cooling, broken disks | AWS handles physical hardware (shared responsibility) |
| **Labels / inventory** | Stickers and spreadsheets | **Tags**, searchable, linked to billing |
| **Experimenting** | Expensive and slow | Launch, test, terminate: costs only a few rupees |
| **When you stop using it** | Hardware sits idle, still paid for | Terminate, billing stops |

**Bottom line:** Traditional IT = **buy and own**. Cloud = **rent and pay only for what you use**, with everything created in minutes through software.

---

## 5. Level 3 – Architect view

### Pricing (purchasing) options

| Option | Discount vs On-Demand | Best for |
|---|---|---|
| **On-Demand** | – | Short-term, unpredictable workloads, testing |
| **Savings Plans** | Up to ~72% | Steady usage, 1- or 3-year commitment (flexible across instance types) |
| **Reserved Instances** | Up to ~72% | Steady usage on a specific instance type, 1 or 3 years |
| **Spot Instances** | Up to ~90% | Fault-tolerant jobs that can be interrupted (batch, big data, CI) |
| **Dedicated Hosts / Instances** | – (premium) | Licensing rules or compliance that need dedicated hardware |

### Design ideas

- **Treat servers as cattle, not pets.** Bake software into an AMI (or use user data), and replace broken instances instead of repairing them by hand.
- **High availability:** an **Auto Scaling group** across **2+ AZs** behind an **Elastic Load Balancer**. (Coming later in the course.)
- **Right-sizing:** start small, watch **CloudWatch** CPU and memory, then change the instance type. Prefer the latest generation and **Graviton (g)** for better price-performance.
- **Storage choice:** important data → **EBS** with snapshots; throwaway fast scratch space → **instance store**; shared files → **EFS**; objects/backups → **S3**.
- **Security:**
  - Never open SSH (22) or RDP (3389) to `0.0.0.0/0`. Use **My IP** or, better, **Session Manager** (no open ports, no keys).
  - Give applications **IAM roles**, never hard-coded access keys.
  - Enforce **IMDSv2** for the metadata service (Day 7: `169.254.169.254`).
  - Reference security groups by **group ID** (e.g. "allow DB port only from the app-server security group") instead of IP ranges.
- **Compliance:** EC2 is validated as **PCI DSS Level 1** compliant, so it can process, store and transmit **credit-card data**. It's still **your** job to configure it securely (shared responsibility).

> 🎯 **Exam tips:**
> - "Data must survive stop/start" → **EBS**, not instance store.
> - "Highest disk speed for temporary cache" → **instance store**.
> - "Cheapest for interruptible batch jobs" → **Spot**.
> - "Steady 24×7 workload for 3 years" → **Savings Plans / Reserved Instances**.
> - "Public IP must not change" → **Elastic IP**.
> - "Launch many identical, pre-configured servers" → **custom AMI**.

---

## 6. Hands-on: launch an instance and spot each feature

When you click **EC2 → Launch instance**, every screen maps to a feature from today:

| Launch screen section | Feature |
|---|---|
| Name and tags | **Tags** |
| Application and OS Images | **AMI** |
| Instance type | **Instance type** |
| Key pair (login) | **Key pair** |
| Network settings (VPC, subnet, auto-assign public IP) | **VPC**, **AZ** (via subnet), public IP |
| Firewall (security groups) | **Security group** |
| Configure storage | **EBS** (and instance store if the type has it) |
| *(after launch)* Elastic IPs menu → Allocate → Associate | **Elastic IP** |

**Practice tasks**
1. Launch `t3.micro` with Amazon Linux 2023 and tag it `Name=day8-web`.
2. Check its **public IP**, then **stop** and **start** it. Has the public IP changed?
3. Allocate an **Elastic IP**, associate it, then stop/start again. Does it stay the same?
4. Log in using **SSH with your key pair**.
5. Clean up: **disassociate and release** the Elastic IP, then **terminate** the instance.

---

## 7. Key terms

| Term | Meaning |
|---|---|
| EC2 | Elastic Compute Cloud – virtual servers on demand |
| Instance | One virtual server |
| AMI | Template with OS + software to launch instances |
| Instance type | Hardware size/family (e.g. t3.micro) |
| EBS | Persistent network disk, one AZ, supports snapshots |
| Instance store | Fast temporary disk, data lost on stop/terminate |
| Snapshot | Point-in-time backup of an EBS volume |
| Key pair | Public key (AWS) + private key (you) for secure login |
| Security group | Stateful virtual firewall, allow rules only |
| VPC | Your private network in AWS |
| Tag | Key–value label on a resource |
| Elastic IP | Static public IPv4 address |
| On-Demand / Spot / Savings Plans | Ways to pay for EC2 |

---

## 8. Quick quiz

1. What does EC2 stand for, and what is an EC2 instance?
2. Which EC2 feature replaces "OS & software" from traditional IT?
3. Which feature replaces the physical firewall?
4. You need storage whose data survives a stop/start. EBS or instance store?
5. In `c7g.xlarge`, what do `c`, `7`, `g` and `xlarge` mean?
6. Who keeps the private key of a key pair?
7. Can a security group have a "deny" rule?
8. Your instance's public IP changes after every restart. What fixes this?
9. Traditional "Labels" map to which AWS feature, and what are they used for?
10. Which pricing option gives the biggest discount for interruptible jobs?

<details>
<summary>Show answers</summary>

1. Elastic Compute Cloud; a virtual server in the AWS Cloud
2. AMI (Amazon Machine Image)
3. Security group
4. EBS
5. c = compute-optimized family, 7 = generation, g = Graviton (ARM) processor, xlarge = size
6. You do. AWS stores only the public key
7. No. Security groups have allow rules only
8. Elastic IP
9. Tags – to name, search, group resources and track costs
10. Spot Instances (up to ~90% off)

</details>

---

## One-line summary

> **EC2 gives you virtual servers on demand: pick an AMI (OS), an instance type (hardware), EBS or instance store (disk), a key pair (login), a security group (firewall), a VPC and AZ (network and location), tags (labels) and an Elastic IP (fixed public address), and pay only while it runs.**
