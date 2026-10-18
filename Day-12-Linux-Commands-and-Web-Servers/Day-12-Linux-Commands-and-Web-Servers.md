# Day 12 – Linux File & Directory Commands + Web Servers on EC2

**Today's agenda**
- Linux commands (continued): directory and file commands, text editors
- Web server setup on a **Windows** instance (IIS)
- Web server setup on a **Linux (Ubuntu)** instance (Apache)

> **Where we are:** Day 11 = moving around Linux (`pwd`, `cd`, `ls`). Today you **create, edit, copy, move and delete** things, then use those skills to publish a real website from EC2. This connects Day 5 (ports), Day 8 (security groups), Day 9 (RDP) and Day 10 (SSH).

---

## PART A – Linux commands (continued)

## 1. Level 1 – Directory (folder) commands

| Task | Command | Example |
|---|---|---|
| Create a directory | `mkdir <name>` (**m**a**k**e **dir**ectory) | `mkdir project` |
| Create a **hidden** directory | `mkdir .<name>` (name starts with a dot) | `mkdir .secret` → visible only with `ls -a` |
| Create nested directories in one go | `mkdir -p a/b/c` | `mkdir -p site/css/img` |
| Rename a directory | `mv <old> <new>` (**m**o**v**e) | `mv project project-v1` |
| Move (relocate) a directory | `mv <name> <destination>/` | `mv project-v1 /tmp/` |
| Copy a directory | `cp -r <source> <destination>` (`-r` = recursive, copies everything inside) | `cp -r site site-backup` |
| Delete an **empty** directory | `rmdir <name>` | `rmdir oldfolder` |
| Delete a directory **and everything inside** | `rm -rf <name>` (`-r` recursive, `-f` force, no questions) | `rm -rf site-backup` |
| Go into / out of a directory | `cd <name>` / `cd ..` | |

> ⚠️ **Clarifications to class notes:**
> - `rmdir` only works on **empty** directories. If the folder has files, use `rm -r` (asks nothing with `-f`).
> - `cd ..` goes to the **parent** directory (one level up). The **previous** directory you were in is `cd -`.

> 🛑 **Danger:** `rm -rf` deletes immediately. There is **no Recycle Bin** in Linux. Double-check the path, and never run `rm -rf /` or `rm -rf *` as root unless you're sure where you are (`pwd` first!).

---

## 2. File commands

### 2.1 Creating files – four ways

| Way | Command | Notes |
|---|---|---|
| **cat** | `cat > notes.txt` → type content → **Ctrl + D** to save | Quick way to type a few lines |
| **touch** | `touch notes.txt` | Creates an **empty** file (or updates its timestamp if it exists) |
| **vi / vim** | `vi notes.txt` | Full editor (see 2.3) |
| **nano** | `nano notes.txt` | Simple, beginner-friendly editor (see 2.4) |

### 2.2 The three `cat` modes

| Command | What it does | Remember as |
|---|---|---|
| `cat notes.txt` | **View** the file | read |
| `cat > notes.txt` | **Replace**: creates the file or **overwrites** everything in it | `>` = wipe and write |
| `cat >> notes.txt` | **Append**: adds to the end, keeps old content | `>>` = add more |

Finish typing with **Ctrl + D**.

```bash
$ cat > fruits.txt
apple
banana
(Ctrl+D)
$ cat >> fruits.txt
mango
(Ctrl+D)
$ cat fruits.txt
apple
banana
mango
```

### 2.3 vi / vim – the classic editor

vi has **modes**. That's what confuses beginners:

```
          vi file.txt
               │
        ┌──────▼──────┐   press  i   ┌──────────────┐
        │ COMMAND mode │ ───────────▶ │ INSERT mode  │  ← type your text here
        │ (starts here)│ ◀─────────── │              │
        └──────┬──────┘   press Esc  └──────────────┘
               │ type :
        ┌──────▼──────┐
        │ :wq  save & quit
        │ :q!  quit WITHOUT saving (force)
        │ :w   save only
        │ :q   quit (only if no changes)
        └─────────────┘
```

**Steps:** `vi index.html` → press **`i`** → type content → press **`Esc`** → type **`:wq`** → **Enter**.

### 2.4 nano – the easy editor

`nano index.html` → type content → **Ctrl + X** → press **Y** (yes, save) → **Enter** (keep the file name).

The shortcuts are shown at the bottom of the screen (`^` means Ctrl).

| | vi / vim | nano |
|---|---|---|
| Difficulty | Harder (modes) | Easy |
| Available | On **every** Linux system | Most systems (may need installing) |
| Save & exit | `Esc` → `:wq` | `Ctrl+X` → `Y` → `Enter` |

### 2.5 Deleting files

| Command | What it does |
|---|---|
| `rm <file>` | Delete a file |
| `rm -i <file>` | Ask for confirmation first (answer **y**) |
| `rm -f <file>` | Force delete, no question |

> 💡 Whether `rm` asks for confirmation depends on the system. On some distributions root has `rm` set to `rm -i` (asks "remove?"), on others (like Ubuntu) it deletes silently. Use `rm -i` if you want to be asked.

### 2.6 Bonus commands you'll need soon

| Command | What it does |
|---|---|
| `cp file1 file2` | Copy a file |
| `mv file1 file2` | Rename/move a file |
| `head -5 file` / `tail -5 file` | First / last 5 lines |
| `tail -f /var/log/syslog` | Watch a log file live (Ctrl+C to stop) |
| `less file` | Scroll through a big file (`q` to quit) |

---

## PART B – Web servers on EC2

## 3. Level 1 – What is a web server?

A **web server** is software that **sends web pages to browsers** when they ask for them.

```
  Your browser ── http://<public-ip> (port 80) ──▶ Security group (port 80 open?)
                                                         │ yes
                                                         ▼
                                                  EC2 instance
                                                  Web server software
                                                  (IIS on Windows / Apache on Linux)
                                                         │
                                                  reads index.html
                                                         │
  Your browser ◀──────────── web page ───────────────────┘
```

| | Windows | Linux (Ubuntu) | Linux (Amazon Linux) |
|---|---|---|---|
| Web server software | **IIS** (Internet Information Services) | **Apache** (`apache2`) | **Apache** (`httpd`) |
| Website folder | `C:\inetpub\wwwroot` | `/var/www/html` | `/var/www/html` |
| Default page | `index.html` / `iisstart.htm` | `index.html` | `index.html` |
| Login method | RDP (3389) | SSH (22) | SSH (22) |

**Step 0 for both:** the security group must allow **HTTP – TCP port 80** from `0.0.0.0/0`. Without it, the browser just times out.

**How to add it:** Instance → **Security** tab → click the security group → **Edit inbound rules** → **Add rule** → Type **HTTP** → Source **Anywhere-IPv4** → **Save rules**.

---

## 4. Lab 1 – Web server on a Windows instance (IIS)

1. **Allow HTTP (port 80)** in the instance's security group (and RDP 3389 for login).
2. Connect with **Remote Desktop** (Day 9).
3. Open **Start → Server Manager**.
4. **Manage → Add Roles and Features** → **Next → Next → Next**.
5. Tick **Web Server (IIS)** → **Add Features** → **Next → Next → Next → Next** → **Install** → **Close**.
6. Open **File Explorer → C: → inetpub → wwwroot**. Delete the existing default files.
7. Create a new file named **`index.html`** with your content, e.g.:

```html
<h1>Hello from my Windows web server on AWS!</h1>
<p>Served by IIS on EC2.</p>
```

8. On your laptop's browser, open **`http://<public-ip>`**. 🎉

> 🛠️ **Common Windows trap:** File Explorer hides extensions, so "index.html" may actually be saved as **`index.html.txt`**. In File Explorer, choose **View → Show → File name extensions** to check, or save from Notepad with "Save as type: **All files**".

**Faster way (PowerShell as Administrator):**

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
Set-Content -Path C:\inetpub\wwwroot\index.html -Value "<h1>Hello from IIS on EC2</h1>"
```

---

## 5. Lab 2 – Web server on an Ubuntu Linux instance (Apache)

1. **Allow HTTP (port 80)** in the security group (and SSH 22 for login).
2. Connect with **MobaXterm** or any SSH method (Day 10).
3. Run:

```bash
sudo -i                          # become root ($ → #)
apt update -y                    # refresh the list of available packages
apt install apache2 -y           # install Apache web server
systemctl start apache2          # start the service now
systemctl enable apache2         # start it automatically after every reboot
systemctl status apache2         # check: should say "active (running)"  (press q to exit)
cd /var/www/html/                # go to the website folder
rm -rf index.html                # delete Apache's default page
nano index.html                  # create your page → type content → Ctrl+X, Y, Enter
```

Example content for `index.html`:

```html
<h1>Hello from my Ubuntu web server on AWS!</h1>
<p>Served by Apache on EC2.</p>
```

4. In your browser, open **`http://<public-ip>`**. 🎉

> ⚠️ **Typo in class notes:** `apt intall` → should be **`apt install`**.

### What each command means

| Command | Meaning |
|---|---|
| `apt` | Ubuntu's **package manager** (app store for the command line) |
| `apt update` | Refresh the package list (do this before installing) |
| `apt install <pkg> -y` | Install a package; `-y` = answer "yes" automatically |
| `systemctl start` | Start a service now |
| `systemctl enable` | Start it automatically at boot |
| `systemctl status` | Show if it's running |
| `systemctl stop` / `restart` | Stop / restart a service |

### Same lab on Amazon Linux 2023

```bash
sudo -i
dnf install httpd -y             # Amazon Linux uses dnf, and Apache is called httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from Amazon Linux</h1>" > /var/www/html/index.html
```

(This is the same thing the **user data** script did automatically in the Day 2 lab.)

---

## 6. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Browser keeps loading, then times out | **Port 80** not open in the security group | Add inbound rule HTTP 80 from 0.0.0.0/0 |
| "This site can't provide a secure connection" | Browser switched to **https://** | Type **`http://`** explicitly. Port 443 isn't set up |
| Apache default page still shows | Old `index.html` still there, or browser cache | Check `ls /var/www/html`; refresh with Ctrl+F5 |
| IIS default page still shows | New file saved as `index.html.txt` | Show file extensions and rename |
| `Unable to locate package apache2` | Skipped `apt update` | Run `apt update -y` first |
| `Permission denied` when editing files | Not root | Use `sudo -i` or `sudo nano ...` |
| Site down after stop/start | Public IP changed | Use the new IP or an **Elastic IP** |
| `systemctl status` shows *failed* | Config error | `journalctl -u apache2` or `/var/log/apache2/error.log` |

---

## 7. Level 3 – Architect view

- **Automate, don't click.** Put the install commands in **user data** so every new instance becomes a web server on boot, or bake them into a **custom AMI**. Manual setup doesn't scale to 10 or 100 servers.
- **One server is a single point of failure.** Production pattern: **Application Load Balancer** → **Auto Scaling group** of web servers across **2+ AZs**. Users hit the load balancer, not an instance IP. (Coming later in the course.)
- **HTTPS (443), not just HTTP (80).** Get a free certificate from **AWS Certificate Manager (ACM)** and attach it to the load balancer or CloudFront.
- **Static websites don't need a server at all.** Plain HTML/CSS/JS sites can be hosted on **Amazon S3 + CloudFront**: cheaper, faster, and nothing to patch.
- **Security:** web servers should allow 80/443 publicly but keep **SSH/RDP closed** (use Session Manager). Keep packages patched (`apt upgrade`, Patch Manager).
- **Logs:** Apache logs are in `/var/log/apache2/` (Ubuntu) or `/var/log/httpd/` (Amazon Linux). IIS logs are in `C:\inetpub\logs\LogFiles`. Ship them to **CloudWatch Logs** with the CloudWatch agent.

> 🎯 **Exam tips:**
> - "Users can't reach the website on EC2" → check the **security group allows 80/443**.
> - "Install software automatically when instances launch" → **user data**.
> - "Host a static website cheaply with no servers" → **S3 static website hosting (+ CloudFront)**.
> - "Distribute traffic across several web servers" → **Elastic Load Balancer**.

---

## 8. Key terms

| Term | Meaning |
|---|---|
| mkdir / rmdir | Make / remove (empty) directory |
| mv | Move or rename |
| cp -r | Copy a directory with everything inside |
| rm -rf | Force-delete a directory and its contents |
| touch | Create an empty file |
| cat > / >> | Overwrite / append to a file |
| vi / vim | Mode-based text editor (`i`, `Esc`, `:wq`, `:q!`) |
| nano | Simple text editor (Ctrl+X, Y, Enter) |
| Web server | Software that serves web pages |
| IIS | Microsoft's web server for Windows |
| Apache (apache2 / httpd) | Popular open-source web server for Linux |
| wwwroot / /var/www/html | Website folders on IIS / Apache |
| apt / dnf | Package managers for Ubuntu / Amazon Linux |
| systemctl | Start, stop, enable and check services |

---

## 9. Quick quiz

1. Command to create a hidden directory called `config`?
2. Command to copy the directory `site` to `site-bak`?
3. `rmdir site` fails with "Directory not empty". What do you use instead?
4. What is the difference between `cat > a.txt` and `cat >> a.txt`?
5. In vi, how do you start typing, and how do you save and quit?
6. How do you quit vi **without** saving?
7. Which port must be open in the security group for a website?
8. Where do you put `index.html` for IIS? For Apache?
9. What is the difference between `systemctl start` and `systemctl enable`?
10. On Ubuntu you get `Unable to locate package apache2`. What did you forget?

<details>
<summary>Show answers</summary>

1. `mkdir .config`
2. `cp -r site site-bak`
3. `rm -r site` (or `rm -rf site` to skip questions)
4. `>` overwrites the file; `>>` adds to the end of it
5. Press `i` to type; press `Esc`, then `:wq` and Enter
6. `Esc`, then `:q!`
7. Port 80 (HTTP), and 443 for HTTPS
8. `C:\inetpub\wwwroot`; `/var/www/html`
9. `start` runs it now; `enable` makes it start automatically at every boot
10. `apt update -y`

</details>

---

## One-line summary

> **Use `mkdir`, `cp -r`, `mv`, `rm -rf`, `touch`, `cat`, `vi` and `nano` to manage files in Linux; to publish a website on EC2, open port 80, install IIS (Windows) or Apache (Linux), and put your `index.html` in `wwwroot` or `/var/www/html`.**
