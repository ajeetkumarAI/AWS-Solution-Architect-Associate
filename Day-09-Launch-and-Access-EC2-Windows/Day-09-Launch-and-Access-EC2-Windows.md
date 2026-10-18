# Day 9 – Operating Systems & Launching / Accessing a Windows EC2 Instance

**Today's agenda**
- What an operating system is, and its types
- Launch a Windows EC2 instance
- Access (log in to) the Windows instance using RDP

> **Where we are:** Day 8 explained every EC2 feature (AMI, instance type, key pair, security group…). Today you **use** them to launch a real Windows server and log in to it.

---

## 1. Level 1 – Basics: What is an operating system?

> **Operating System (OS):** an **interface between the user and the hardware**.

```
        You (user)
            │
     ┌──────▼──────┐
     │     OS      │  ← translates your clicks/commands into hardware actions
     └──────▲──────┘
            │
   Hardware (CPU, RAM, disk, network)
```

**Simple example:** A **driver** in a taxi. You say "take me to the station"; the driver handles the steering, brakes and gears. The OS handles the hardware for you.

### Types of OS – by how you use it

| Type | Full form | How you use it | Examples |
|---|---|---|---|
| **GUI** | Graphical User Interface | Windows, icons, mouse clicks | Windows, macOS |
| **CLI** | Command-Line Interface | Typing commands | Linux (on servers) |

> 💡 **Clarification:** This is about the *usual* way each OS is used, not a hard rule. Linux has full desktop GUIs (Ubuntu Desktop), and Windows has strong command lines (PowerShell, Command Prompt), and even a no-GUI edition called **Windows Server Core**. On cloud servers, Linux is almost always used through the CLI.

### Types of OS – by purpose

| Type | Meant for | Examples |
|---|---|---|
| **Client OS** | End-user devices: laptops, desktops | Windows 10, Windows 11 |
| **Network (Server) OS** | Server-based or service-based configuration: hosting websites, databases, Active Directory, file sharing | Windows Server 2025, 2022 |

### Default owner (super-user) account

| OS | Default owner | Notes |
|---|---|---|
| **Windows** | **Administrator** | Used to log in to Windows EC2 instances |
| **Linux** | **root** | On EC2 you don't log in as root directly. You use a default user (`ec2-user` for Amazon Linux, `ubuntu` for Ubuntu) and run admin commands with **`sudo`** |

---

## 2. Windows EC2 – what you need to know before launching

| Item | Value |
|---|---|
| **OS / AMI** | Microsoft Windows Server (choose version from dropdown) |
| **Username** | **Administrator** |
| **Connection method (channel)** | **RDP** – Remote Desktop Protocol |
| **Port** | **3389** (RDP) |
| **Instance type (hardware)** | **t2.micro / t3.micro** for practice |

**Linux vs Windows on EC2 (side by side)**

| | Linux instance | Windows instance |
|---|---|---|
| Login method | SSH | RDP |
| Port | 22 | 3389 |
| Username | ec2-user / ubuntu | Administrator |
| How the key pair is used | Key **is** the login | Key **decrypts the password** |
| Screen | Terminal (text) | Full Windows desktop |
| Cost | Lower | Higher (Windows licence is included in the hourly price) |

> 🔗 **Connect the dots:** Port numbers (Day 5), public IP (Day 7), AMI, instance type, key pair and security group (Day 8): all of them come together today.

---

## 3. Level 2 – Lab: Launch a Windows EC2 instance

1. Log in to the **AWS Console**. Check the **Region** (top right), e.g. **Mumbai (ap-south-1)**.
2. In the search bar type **EC2** → open **EC2**.
3. In the left menu click **Instances** → **Launch instance**.
4. **Name and tags:** `My-Windows-Server` *(this creates the `Name` tag)*.
5. **Application and OS Images (AMI):** choose **Windows**, then the version from the dropdown (e.g. *Microsoft Windows Server 2025 Base*).
6. **Instance type:** **t2.micro** or **t3.micro** only.
7. **Key pair (login):** **Create new key pair** →
   - Name: e.g. `windows-key`
   - Key pair type: **RSA** *(required for Windows password decryption)*
   - Private key file format: **.pem**
   - **Create key pair** → the `.pem` file downloads. **Keep it safe; AWS doesn't keep a copy.**
8. **Network settings:** don't change anything (default VPC, auto-assign public IP = Enable).
9. **Firewall (security groups):** make sure **Allow RDP traffic** is ticked.
   - Better practice: change the source from *Anywhere (0.0.0.0/0)* to **My IP**.
10. **Configure storage:** don't change anything (default 30 GiB gp3 root volume for Windows).
11. Check the **Summary** → click **Launch instance**.
12. Go to **Instances** and wait until **Instance state = Running** and **Status check = 2/2 checks passed**.

---

## 4. Lab: Access the Windows instance

### Step A – Get the Administrator password

1. Open the **Instance ID** → click **Connect**.
2. Select the **RDP client** tab.
3. Click **Get password** → **Upload private key file** (your `.pem`) → **Decrypt password**.
4. **Copy and note the password.**

> ⏳ If it says *"Password not available yet"*, wait **about 4–5 minutes** after launch. Windows is still setting up.

```
 Your .pem (private key) ──decrypts──▶ Administrator password
                                         (AWS encrypted it with your public key)
```

### Step B – Connect using Remote Desktop Connection (built into Windows)

1. On your laptop press **Win + R** → type `mstsc` → Enter. *(Or search "Remote Desktop Connection".)*
2. **Computer:** paste the instance's **Public IPv4 address** or **Public IPv4 DNS**.
3. Click **Show Options** → **User name:** `Administrator`.
4. Click **Connect** → enter the **password** from Step A.
5. Accept the certificate warning (**Yes**).
6. 🎉 You now see the Windows Server desktop, running in an AWS data center.

**Tip:** On the Connect page you can also click **Download remote desktop file** (`.rdp`). Double-click it to open RDP with the address already filled in.

### Step C – Or connect using MobaXterm (third-party app)

1. Open **MobaXterm** → **Session** → **RDP**.
2. **Remote host:** public IP.
3. **Specify username:** `Administrator`.
4. Click **OK** → enter the password.

> ⚠️ **Correction to class notes:** The notes say *"Windows supports only one single remote connection at a time."* That's true for **client** Windows (10/11). **Windows Server** allows **two** simultaneous administrative RDP sessions by default. More users need **Remote Desktop Services (RDS) licences**.

### Step D – Clean up (very important)

- **Instance state → Terminate instance.**
- Windows instances cost more than Linux, so don't leave them running.
- Also delete unused key pairs and security groups if you created extras.

---

## 5. Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| RDP says "can't connect" / times out | Security group doesn't allow **3389** from your IP | Edit inbound rules → RDP, TCP 3389, source **My IP** |
| Worked yesterday, fails today | Your home/office **public IP changed** | Update the security group rule to your new IP |
| Wrong address after restart | Instance was **stopped/started**; public IP changed | Use the new public IP, or attach an **Elastic IP** (Day 8) |
| "Password not available yet" | Windows is still initializing | Wait ~4–5 minutes |
| Can't decrypt password | Wrong `.pem` file, or key type was ED25519 | Use the same key pair chosen at launch; use **RSA** for Windows |
| Desktop very slow | t2/t3.micro has only **1 GiB RAM** | Fine for practice; real servers use t3.medium or larger |
| Company network blocks RDP | Firewalls often block port 3389 | Use **Fleet Manager** (browser RDP) – see section 6 |

---

## 6. Level 3 – Architect view

- **Never leave RDP (3389) open to `0.0.0.0/0`.** It's one of the most scanned and attacked ports on the internet. Options, from good to best:
  1. Security group source = **My IP** (`x.x.x.x/32`).
  2. Connect through a **bastion host** or **VPN**.
  3. **AWS Systems Manager**: **Fleet Manager Remote Desktop** (RDP in the browser) or **Session Manager port forwarding**. **No open inbound ports and no public IP needed**, and every session is logged.
- **Licensing:** Windows AMIs are **License Included**: the Windows Server licence is part of the hourly price (this is why Windows costs more). **BYOL** (Bring Your Own License) usually needs **Dedicated Hosts**.
- **Sizing:** Windows needs more RAM than Linux. Typical production starting points are **t3.medium / m6i.large**. Use **CloudWatch** to right-size.
- **Patching & setup:** AWS publishes updated Windows AMIs every month. Use **Systems Manager Patch Manager** for patching and **EC2Launch v2** / user data (PowerShell) for first-boot setup.
- **Active Directory:** join Windows instances to **AWS Managed Microsoft AD** (AWS Directory Service) so users log in with company accounts instead of a shared Administrator password.
- **Change the Administrator password** after first login, or better, use domain accounts.

> 🎯 **Exam tips:**
> - "Connect to Windows instance" → **RDP, port 3389**; Linux → **SSH, port 22**.
> - "Administrator password for Windows EC2" → **decrypt using the key pair's private key**.
> - "Access instances without opening inbound ports / without bastion" → **Systems Manager Session Manager / Fleet Manager**.
> - "Use existing Windows Server licences" → **BYOL on Dedicated Hosts**.

---

## 7. Key terms

| Term | Meaning |
|---|---|
| Operating System | Interface between user and hardware |
| GUI / CLI | Graphical vs command-line way of using an OS |
| Client OS | OS for end-user devices (Windows 10/11) |
| Network / Server OS | OS for servers and services (Windows Server 2025) |
| Administrator | Default owner account on Windows |
| root | Default owner account on Linux |
| sudo | Run a Linux command with root (admin) rights |
| RDP | Remote Desktop Protocol – remote access to a Windows desktop |
| Port 3389 | RDP port |
| mstsc | Windows' built-in Remote Desktop Connection app |
| MobaXterm | Third-party tool for RDP and SSH |
| Public IPv4 DNS | Name form of the public IP, e.g. `ec2-13-233-x-x.ap-south-1.compute.amazonaws.com` |
| Fleet Manager | AWS Systems Manager feature for RDP in the browser |

---

## 8. Quick quiz

1. Define an operating system in one line.
2. Give one example each of a client OS and a network (server) OS.
3. What are the default owner accounts on Windows and Linux?
4. Which protocol and port are used to connect to a Windows EC2 instance?
5. How do you get the Administrator password for a Windows EC2 instance?
6. Which key pair type should you choose for Windows instances?
7. Your RDP connection times out. What is the first thing to check?
8. How many simultaneous admin RDP sessions does Windows Server allow by default?
9. Why is a Windows instance more expensive than a Linux one of the same size?
10. What must you do after finishing practice?

<details>
<summary>Show answers</summary>

1. An interface between the user and the hardware
2. Client: Windows 10/11; Server: Windows Server 2025
3. Windows: Administrator; Linux: root
4. RDP, port 3389
5. Instance → Connect → RDP client → Get password → upload .pem → Decrypt
6. RSA
7. The security group inbound rule for TCP 3389 from your current IP
8. Two
9. The Windows Server licence is included in the hourly price
10. Terminate the instance

</details>

---

## One-line summary

> **To use a Windows EC2 instance: launch it from a Windows AMI with an RSA key pair and RDP (3389) allowed, decrypt the Administrator password with your .pem file, connect with Remote Desktop, and terminate it when you're done.**
