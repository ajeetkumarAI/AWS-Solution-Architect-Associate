# Day 10 – Launch & Access a Linux EC2 Instance

**Today's agenda**
- Launch and access Linux EC2 instances
- Download PuTTY and MobaXterm for Linux access

> **Where we are:** Day 9 = Windows instance through **RDP (3389)**. Today = Linux instance through **SSH (22)**. Most servers in the cloud run Linux, so this is the method you'll use most.

---

## 1. Level 1 – Basics: Linux EC2 at a glance

| Item | Value |
|---|---|
| **OS / AMI** | Amazon Linux or Ubuntu Linux |
| **Username** | `ec2-user` (Amazon Linux) / `ubuntu` (Ubuntu) |
| **Connection method (channel)** | **SSH** – Secure Shell |
| **Port** | **22** |
| **Instance type (hardware)** | t2.micro / t3.micro |
| **Login proof** | Your key pair's **private key** (`.pem` or `.ppk`). No password |

> ⚠️ **Corrections to class notes:**
> - The notes say *"Port: RDP: 22"*. It should be **SSH: 22**. RDP is 3389 (Windows, Day 9).
> - The notes say *"default user is root"*. Every Linux system has a **root** (super-user) account, but on EC2 you **can't log in as root directly**. You log in as `ec2-user`/`ubuntu` and switch to root with `sudo` (Day 11).

**Simple example:** SSH is like a **secure phone line** into the server. Instead of a desktop screen (RDP), you get a **text window** where you type commands.

### Windows vs Linux access (recap)

| | Windows (Day 9) | Linux (Day 10) |
|---|---|---|
| Protocol | RDP | SSH |
| Port | 3389 | 22 |
| User | Administrator | ec2-user / ubuntu |
| Key pair used to | Decrypt the password | **Log in directly** |
| Screen | Desktop (GUI) | Terminal (CLI) |
| Users at the same time | 2 admin sessions (Windows Server) | **Many users at a time** |

---

## 2. Lab: Launch a Linux EC2 instance

1. AWS Console → check Region (e.g. **Mumbai ap-south-1**) → search **EC2** → **Instances** → **Launch instance**.
2. **Name:** `My-Linux-Server`.
3. **AMI:** **Amazon Linux 2023** (or **Ubuntu Server 24.04 LTS**).
4. **Instance type:** **t2.micro / t3.micro**.
5. **Key pair:** create new → name `linux-key` → type **RSA** or **ED25519** → format:
   - **.pem** → for Command Prompt, PowerShell, Linux terminal, MobaXterm
   - **.ppk** → for PuTTY (or convert later with PuTTYgen)
6. **Network settings:** leave defaults. Make sure **Allow SSH traffic** is ticked (better: source **My IP**).
7. **Storage:** leave the default (8 GiB gp3 for Amazon Linux).
8. **Launch instance** → wait for **Running** and **2/2 checks passed**.

---

## 3. Level 2 – Five ways to access a Linux instance

| # | Method | Needs | Best for |
|---|---|---|---|
| 1 | **EC2 Instance Connect** | Only a browser | Quick access, no key file needed |
| 2 | **Windows Command Prompt / PowerShell** | `.pem` + built-in `ssh` | Windows 10/11 users |
| 3 | **Linux / macOS terminal** | `.pem` + `ssh` | Linux or Mac users |
| 4 | **MobaXterm** (free Home edition) | `.pem` | Windows users who want tabs, file transfer and a GUI |
| 5 | **PuTTY** (`putty.exe` + `puttygen.exe`) | `.ppk` | Classic Windows SSH tool |

### Method 1 – EC2 Instance Connect (browser)

1. Select the instance → **Connect**.
2. Open the **EC2 Instance Connect** tab.
3. Check the **public IP** and **username** (`ec2-user` / `ubuntu`).
4. Click **Connect** → a terminal opens in a new browser tab.

> 💡 Works out of the box on Amazon Linux and Ubuntu AMIs. The security group must allow **port 22** from the EC2 Instance Connect service (with "My IP" only, the browser method may fail; use your terminal in that case).

### Method 2 – Windows Command Prompt / PowerShell

1. Instance → **Connect** → **SSH client** tab. Copy the example command shown there.
2. Open **Command Prompt** and go to the folder that has your `.pem` file:

```bat
cd %USERPROFILE%\Downloads
dir *.pem
```

3. If the `.pem` file isn't there, copy it into that folder and run `dir` again.
4. Connect:

```bat
ssh -i "linux-key.pem" ec2-user@13.233.45.67
```

5. Type **yes** when asked *"Are you sure you want to continue connecting?"* and press Enter.

```
 ssh  -i linux-key.pem   ec2-user   @   13.233.45.67
  │    │      │              │             │
 tool  identity file      username       public IP (or public DNS)
```

> 🛠️ **If you see "WARNING: UNPROTECTED PRIVATE KEY FILE"** on Windows, the `.pem` file is readable by other users. Fix it in PowerShell:
> ```powershell
> icacls .\linux-key.pem /inheritance:r
> icacls .\linux-key.pem /grant:r "$($env:USERNAME):(R)"
> ```

### Method 3 – Linux / macOS terminal

```bash
cd ~/Downloads             # folder with the .pem file
chmod 400 linux-key.pem    # only you can read the key; ssh refuses keys that others can read
ssh -i linux-key.pem ubuntu@13.233.45.67
# type: yes
```

`chmod 400` = owner can **read**, nobody else can do anything. More on permissions in later classes.

### Method 4 – MobaXterm

1. Open MobaXterm → **Session** → **SSH**.
2. **Remote host:** public IP → tick **Specify username** → `ec2-user` / `ubuntu`.
3. **Advanced SSH settings** → tick **Use private key** → **Browse** → select the `.pem` file → **Open**.
4. Click **OK**.

Bonus: the left panel shows the server's files, so you can **drag and drop** files to and from the instance.

### Method 5 – PuTTY

PuTTY doesn't read `.pem` files, so first **convert** it to `.ppk` with PuTTYgen.

**Step A – Convert with PuTTYgen**
1. Open **puttygen.exe** → **Load**.
2. Change the file filter (bottom right) to **All Files (\*.\*)** → select the `.pem` → **Open** → **OK**.
3. Click **Save private key** → (Yes to "save without passphrase" for practice) → give it a name → **Save** → you get `linux-key.ppk`.

**Step B – Connect with PuTTY**
1. Open **putty.exe**.
2. **Host Name (or IP address):** public IP. Check **Port 22** and **Connection type: SSH**.
3. Left menu: **Connection → SSH → Auth → Credentials** → **Browse** → select the `.ppk` → **Open**.
4. Click **Open** → **Accept** the security alert.
5. **login as:** `ec2-user` / `ubuntu`.

> 💡 **Tip:** In PuTTY, go back to **Session**, type a name under *Saved Sessions* and click **Save** before connecting, so next time it's one double-click.

### Clean up
**Terminate** the instance after practice.

---

## 4. Troubleshooting

| Error / symptom | Likely cause | Fix |
|---|---|---|
| `Connection timed out` | Security group doesn't allow **22** from your IP, or wrong IP | Add inbound rule SSH, TCP 22, source **My IP**; check the current public IP |
| `Permission denied (publickey)` | Wrong **username** or wrong **key** | Amazon Linux → `ec2-user`; Ubuntu → `ubuntu`; use the key chosen at launch |
| `UNPROTECTED PRIVATE KEY FILE` | Key file permissions too open | `chmod 400` (Linux/Mac) or `icacls` (Windows) |
| `No such file or directory` for the .pem | You're in the wrong folder | `cd` to the folder; check with `dir` / `ls` |
| Worked yesterday, IP different today | Instance was stopped/started | Use the new IP, or attach an **Elastic IP** |
| PuTTY "unable to use key file" | Gave PuTTY the `.pem` | Convert to `.ppk` with PuTTYgen |
| `REMOTE HOST IDENTIFICATION HAS CHANGED` | A new instance reused an old IP | Remove the old entry: `ssh-keygen -R <ip>` |

---

## 5. Level 3 – Architect view

- **Port 22 open to the world is a risk.** It's constantly scanned by bots. Restrict to **My IP**, or remove it completely and use:
  - **AWS Systems Manager Session Manager** – shell in the browser or CLI, **no port 22, no key pair, no public IP**, every session logged to CloudWatch/S3. This is the preferred method in companies.
  - **EC2 Instance Connect Endpoint** – SSH into instances in **private subnets** without a bastion or public IP.
- **Bastion (jump) host:** one hardened public instance used to reach private instances. Being replaced by the two options above.
- **Key management:** one key per environment/team, never share `.pem` files in chat or email, and never commit them to Git. Lost the key? Use Session Manager or the "replace key via EBS volume" recovery method.
- **ED25519 vs RSA:** ED25519 keys are shorter and modern; Windows instances still need RSA (Day 9).
- **Automate setup** with **user data** (Day 2 lab) and custom **AMIs** so you rarely need to SSH in at all. Ideal production servers are never logged into by hand.

> 🎯 **Exam tips:**
> - "Connect to Linux instance" → **SSH, port 22**, key pair.
> - "Remove the need for SSH keys / bastion hosts and audit all sessions" → **Session Manager**.
> - "Connect to an instance in a private subnet without a public IP" → **Session Manager** or **EC2 Instance Connect Endpoint**.

---

## 6. Key terms

| Term | Meaning |
|---|---|
| SSH | Secure Shell – encrypted remote command-line access |
| Port 22 | SSH port |
| ec2-user / ubuntu | Default login users on Amazon Linux / Ubuntu |
| .pem | Private key format for OpenSSH, MobaXterm |
| .ppk | Private key format for PuTTY |
| PuTTYgen | Tool to convert .pem ↔ .ppk |
| chmod 400 | Make the key file readable only by you |
| EC2 Instance Connect | Browser-based SSH from the AWS console |
| Session Manager | Keyless, portless shell access through AWS Systems Manager |
| Bastion host | Public "jump" server used to reach private servers |

---

## 7. Quick quiz

1. Which protocol and port do you use to connect to a Linux EC2 instance?
2. What is the default username on Amazon Linux? On Ubuntu?
3. Write the SSH command to connect to `13.233.45.67` as `ec2-user` using `linux-key.pem`.
4. Why do we run `chmod 400` on the key file?
5. Which file format does PuTTY need, and which tool creates it?
6. You get `Permission denied (publickey)`. Name two possible causes.
7. You get `Connection timed out`. What should you check first?
8. Which access method needs only a web browser?
9. Can many users log in to a Linux machine at the same time?
10. *(Architect)* Which service lets you connect with no SSH port open and no key pair?

<details>
<summary>Show answers</summary>

1. SSH, port 22
2. `ec2-user`; `ubuntu`
3. `ssh -i linux-key.pem ec2-user@13.233.45.67`
4. SSH refuses private keys that other users can read; 400 = read-only for the owner
5. `.ppk`; PuTTYgen
6. Wrong username, or wrong key file
7. The security group inbound rule for port 22 from your IP (and that you're using the current public IP)
8. EC2 Instance Connect
9. Yes
10. AWS Systems Manager Session Manager

</details>

---

## One-line summary

> **Linux EC2 instances are accessed with SSH on port 22 using your private key: from the browser (Instance Connect), Command Prompt/PowerShell, a Linux terminal, MobaXterm (.pem) or PuTTY (.ppk).**
