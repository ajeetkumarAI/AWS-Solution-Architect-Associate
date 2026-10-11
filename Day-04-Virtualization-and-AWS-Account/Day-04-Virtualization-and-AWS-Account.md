# Day 4 – Virtualization & AWS Account Creation

**Today's agenda**
- Understanding virtualization
- Creating an AWS account

---

## 1. Why do we need virtualization? (Netflix example)

```
   ┌────────────┐        ┌────────────┐   (3)   ┌─────────────────┐
   │ Web Server │        │ App Server │─────────│ Database Server │
   └─────▲──────┘        └─────▲──────┘         └─────────────────┘
         │ (1)                 │ (2)
         │                     │
         └──────── Mobile ─────┘
                     │
                    User
```

1. **Download & install** the app → from the **Web server**
2. **Register / configure** → through the **App server**
3. **User data** is stored in the **Database**, which connects directly to the **App server**

### Server configuration (each server)

| Item | Value |
|---|---|
| CPU | 16 cores |
| RAM | 16 GB |
| Disk | 500 GB SSD |
| Price | ₹7 lakh × 3 servers = **₹21 lakh** |

### Why not put all 3 on one server?
As infrastructure people, we keep them separate because of:
- **Security issues** – one breach exposes everything
- **Hardware management issues**
- **Service failure** – one crash takes down all three

### The waste problem

| Server | Usage |
|---|---|
| Web server | 10% |
| App server | 30% |
| DB server | 10% |
| **Total** | **50% used out of 300% capacity** |

```
Web  [█░░░░░░░░░] 10%
App  [███░░░░░░░] 30%
DB   [█░░░░░░░░░] 10%
All  [██░░░░░░░░] ~17%  ← only one-sixth of what we paid for is used
```

We paid ₹21 lakh but use only about **1/6** of the hardware. Roughly **₹17.5 lakh sits idle**.

**Solution → Virtualization:** run all three as separate virtual machines on **one** physical server — still isolated, but far less waste.

---

## 2. Level 1 – Basics: What is virtualization?

> **Definition:** **One physical machine acting as multiple virtual machines.**

**Simple example:** One big hall divided by movable walls into three rooms. Each group feels it has its own room, but it is one building.

- **Hypervisor** – the software that splits the physical machine
- **Virtual Machine (VM)** – a "computer inside a computer" with its own OS and apps
- **Guest OS** – the operating system inside a VM
- **Host** – the real physical machine

---

## 3. Level 2 – Going deeper: Two types of virtualization

### Type 2 – Hosted (the diagram from class)

```
┌──────────── Physical Server ────────────┐
│   ┌── VM1 ──┐   ┌── VM2 ──┐             │
│   │  App    │   │  App    │             │
│   │ GuestOS │   │ GuestOS │             │
│   │  VHW    │   │  VHW    │  VHW = Virtual Hardware
│   └─────────┘   └─────────┘             │
│      Virtualization technique           │
│      Host Operating System   ⚠️         │
│      Hardware                           │
└─────────────────────────────────────────┘
```

**Problem:** If the **Host OS** has bugs or runs slow, it **hurts the performance of every VM**.

### Type 1 – Bare-metal / Hardware virtualization (what clouds use)

```
                 ┌── vm1
   [ HW ] ── ( V ) 
                 └── vm2
   HW = Hardware,  V = Hypervisor (runs directly on hardware, no host OS)
```

To avoid the host-OS problem, the industry moved to **hardware virtualization**: the hypervisor sits **directly on the hardware**. It is faster and more secure.

| | Type 2 (Hosted) | Type 1 (Bare-metal) |
|---|---|---|
| Runs on | A host OS | Directly on hardware |
| Performance | Lower | Higher |
| Used for | Laptops, testing | Data centers, cloud |
| Examples | VMware Workstation, VirtualBox | VMware ESXi, Microsoft Hyper-V, AWS Nitro |

### Virtualization products

| Vendor | Products | What they are |
|---|---|---|
| **VMware** | vSphere, vCenter, ESXi, Workstation | **ESXi** = bare-metal hypervisor; **vCenter** = manages many ESXi hosts; **vSphere** = full suite; **Workstation** = desktop (Type 2) |
| **Microsoft** | Hyper-V | Hypervisor built into Windows Server / Windows Pro |
| **AWS** | Xen → **Nitro** | Early EC2 ran on Xen; Nitro launched in **2017** and now runs almost all instance types |

> ⚠️ **Correction to class notes:** The notes say "Xen till 2019, Nitro from 2019". AWS actually launched Nitro in **November 2017** (with C5 instances). Xen-based instance types were phased out **gradually** after that.

> 🔗 **Connect the dots:** Every **EC2 instance** you launch is a **virtual machine** running on the Nitro hypervisor in an AWS data center. Virtualization is what makes "renting part of a server by the hour" possible.

---

## 4. Level 3 – Architect view

- Modern virtualization uses CPU **hardware extensions** (Intel VT-x, AMD-V) and nested paging (EPT/NPT), so VMs run at near-native speed.
- **AWS Nitro System** moves networking, storage (EBS) and management onto dedicated **Nitro Cards**, leaving a very small hypervisor. Result: almost all host CPU/RAM goes to customers, and the attack surface is small.
- Nitro has **no operator shell access** to hosts – AWS staff cannot log in to the machine running your instance.
- **Nitro Enclaves** – isolated environments inside an instance for highly sensitive data (keys, card data).
- **Bare-metal instances** (e.g. `*.metal`) give you the physical server with no hypervisor — useful for licensing or special workloads.
- **Containers vs VMs:** VMs virtualize **hardware** (each has its own OS); containers (Docker) virtualize the **OS** (share the host kernel) – lighter and faster to start. You'll meet ECS, EKS and Fargate later.

---

## 5. Creating an AWS account – step by step

**You need:** email address, mobile number, identity document, payment mode.

- [ ] Go to **www.aws.amazon.com** → click **Create an AWS Account**
- [ ] Enter **email** and **account name** → verify the code sent to your email
- [ ] Set a strong **root user password**
- [ ] Fill **contact details** → verify your **mobile number** (SMS/call)
- [ ] Provide your **identity document** when asked
- [ ] Add a **payment method** (card or UPI)
- [ ] Choose the free **Basic support** plan → finish

### 🔐 Do these immediately after sign-up

- [ ] Turn on **MFA** (multi-factor authentication) for the **root user**
- [ ] Create a **budget alert** in Billing → *Budgets* (e.g. alert at $5 — AWS budgets are set in USD)
- [ ] Create an **IAM user** (or IAM Identity Center user) for daily work
- [ ] **Stop using root** for everyday tasks; never create access keys for root
- [ ] Set your default Region to **Mumbai (ap-south-1)**

> 🎯 **Exam tip:** Root-user questions almost always want: **enable MFA**, **don't create root access keys**, and **use IAM identities for daily work**.

---

## 6. Key terms

| Term | Meaning |
|---|---|
| Virtualization | One physical machine acting as multiple VMs |
| Hypervisor | Software that creates and runs VMs |
| VM | Virtual machine with its own OS and apps |
| Guest OS | OS running inside a VM |
| Host OS | OS on the physical machine (Type 2 only) |
| VHW | Virtual hardware given to each VM |
| Type 1 hypervisor | Runs directly on hardware (ESXi, Hyper-V, Nitro) |
| Type 2 hypervisor | Runs on top of a host OS (Workstation, VirtualBox) |
| Nitro | AWS's current hypervisor system |
| Root user | The account's email login with full power |
| MFA | Second login check (app code) in addition to password |

---

## 7. Quick quiz

1. Define virtualization in one line.
2. What is the main problem with the host-OS (Type 2) approach?
3. Which hypervisor runs most EC2 instances today?
4. Netflix example: web 10%, app 30%, DB 10% on three servers. What share of total capacity is used?
5. Name three things you should do right after creating an AWS account.
6. Is VMware ESXi Type 1 or Type 2?

<details>
<summary>Show answers</summary>

1. One physical machine acting as multiple virtual machines.
2. Bugs or slowness in the host OS affect every VM.
3. AWS Nitro
4. 50 out of 300 = about 17% (one-sixth)
5. Enable MFA on root, set a budget alert, create an IAM user and stop using root
6. Type 1 (bare-metal)

</details>

---

## One-line summary

> **Virtualization lets one physical server run many isolated virtual machines, cutting waste. AWS EC2 is built on it, using the Nitro hypervisor.**
