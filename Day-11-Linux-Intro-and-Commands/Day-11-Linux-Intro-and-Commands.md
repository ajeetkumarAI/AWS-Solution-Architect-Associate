# Day 11 – Linux Introduction & Basic Navigation Commands

**Today's agenda**
- Linux introduction
- Basic Linux navigation commands: switching users, directories, listing, clearing the screen, history

> **Where we are:** Day 10 got you logged in to a Linux EC2 instance. Today you learn to **move around** inside it. Practise every command below on your own instance.

---

## 1. Level 1 – Basics: Why Linux?

- **Linux** is a free, open-source operating system (Day 9: OS = interface between user and hardware).
- It powers **most servers on the internet** and most EC2 instances, because it's free, stable, secure and light.
- On servers it's mostly used through the **CLI** (command line): you type commands instead of clicking.

**Popular Linux distributions ("flavours") and their default EC2 users**

| Distribution | Default EC2 login user |
|---|---|
| Amazon Linux | `ec2-user` |
| Ubuntu | `ubuntu` |
| RHEL (Red Hat) | `ec2-user` |
| SUSE | `ec2-user` |
| Debian | `admin` |
| Fedora | `fedora` |
| **Every Linux system** | also has the super-user **`root`** (full power) |

> ⚠️ **Correction to class notes:** The notes say RHEL/Fedora/Debian/SUSE use "root/ec2-user". On EC2 you never log in directly as root. RHEL and SUSE use `ec2-user`, Debian uses `admin`, Fedora uses `fedora`. Root is reached afterwards with `sudo`.

---

## 2. Understanding the prompt

When you log in you see a **prompt** like this:

```
ubuntu@ip-172-31-11-229:~$
  │          │          │ │
  │          │          │ └─ $ = standard (normal) user   (# = root user)
  │          │          └─── current directory (~ = your home folder)
  │          └────────────── hostname (built from the instance's PRIVATE IP 172.31.11.229)
  └───────────────────────── username
```

| Symbol at the end | Meaning |
|---|---|
| **`$`** | You are a **standard user** |
| **`#`** | You are the **root user** (be careful!) |

**Class example:** `ubuntu@ip-172-31-11-229:/root$`
- `ubuntu` → username
- `ip-172-31-11-229` → device identity (hostname from the private IP, Day 7)
- `/root` → the **current directory** (here, the root user's home folder)
- `$` → standard user

---

## 3. Switching users

| Command | What it does |
|---|---|
| `sudo su` | Become **root**, staying in the current directory |
| `sudo su -` | Become **root** with a fresh root login (goes to `/root`) |
| `sudo -i` | Same as `sudo su -` (recommended form) |
| `sudo su <username>` | Switch to another user, e.g. `sudo su ec2-user` |
| `exit` or **Ctrl + D** | Go back to the previous user / log out |
| `whoami` | Show which user you are right now |

**`sudo`** = "**s**uper-**u**ser **do**": run something with root (admin) power.

```bash
ubuntu@ip-172-31-11-229:~$ whoami
ubuntu
ubuntu@ip-172-31-11-229:~$ sudo -i
root@ip-172-31-11-229:~# whoami        # note: $ became #
root
root@ip-172-31-11-229:~# exit           # or Ctrl+D
ubuntu@ip-172-31-11-229:~$
```

**Simple example:** You are an employee (`$`). `sudo` is like borrowing the manager's master key (`#`) for a task. Hand it back (`exit`) when done.

---

## 4. Linux directories – the three that confuse everyone

Linux files are arranged like an **upside-down tree** starting at **`/`**.

```
/                    ← "root directory" – the TOP of everything (default/starting location)
├── root/            ← home folder of the ROOT USER   (/root)
├── home/
│   ├── ubuntu/      ← home folder of user ubuntu     (/home/ubuntu)  = ~ when logged in as ubuntu
│   └── ec2-user/    ← home folder of user ec2-user   (/home/ec2-user)
├── etc/             ← configuration files
├── var/             ← logs (/var/log), websites (/var/www)
├── tmp/             ← temporary files
└── bin, usr, ...    ← programs and commands
```

| Name | Path | Symbol | Meaning |
|---|---|---|---|
| **Root directory** | `/` | `/` | The top of the whole file system |
| **Home directory** | `/home/<user>` | `~` (tilde) | Your personal folder |
| **Root user's home** | `/root` | `~` *when you are root* | Home folder of the root user |

> ⚠️ **Clarification:** The notes list *"Root directory (~)(root user)"*. The word "root" is used for **two different things**:
> - **`/`** is the **root directory**, the top of the file system.
> - **`/root`** is the **root user's home directory**.
>
> `~` always means "the **current user's** home": `/home/ubuntu` for ubuntu, `/root` for root.

---

## 5. Navigation commands

| Command | Full form | What it does | Example |
|---|---|---|---|
| `pwd` | **p**resent **w**orking **d**irectory | Shows where you are | `pwd` → `/home/ubuntu` |
| `cd <name>` | **c**hange **d**irectory | Go into a folder | `cd /var/log` |
| `cd ..` | | Go **up** one level (parent folder) | `/var/log` → `/var` |
| `cd /` | | Go to the **root directory** | |
| `cd ~` or `cd` | | Go to **your home** | |
| `cd -` | | Go back to the **previous** folder | |
| `sudo su -` | | Become root **and** land in `/root` | |

```bash
$ pwd
/home/ubuntu
$ cd /
$ pwd
/
$ cd var
$ cd log
$ pwd
/var/log
$ cd ..
$ pwd
/var
$ cd ~
$ pwd
/home/ubuntu
```

**Absolute vs relative paths**
- **Absolute** – starts with `/`, works from anywhere: `cd /var/log`
- **Relative** – starts from where you are: from `/var`, just `cd log`

---

## 6. List commands

| Command | What it shows |
|---|---|
| `ls` | Names of files and folders in the current directory |
| `ls -a` | **All** items, including **hidden** ones (names starting with `.`, like `.bashrc`) |
| `ll` | **Long list** view: permissions, owner, size, date |
| `ll -a` or `ls -la` | Long list **including hidden** items |
| `ls /etc` | List another directory without going into it |

> 💡 `ll` is a **shortcut (alias)** for `ls -l`, set up on Amazon Linux and Ubuntu. On some systems it doesn't exist, so use `ls -l` / `ls -la`, which work everywhere.

### Reading `ls -l` output

```
drwxr-xr-x  2  ubuntu  ubuntu  4096  Oct 18 10:15  notes
-rw-r--r--  1  ubuntu  ubuntu   220  Oct 18 10:15  .bash_logout
│└──┬────┘     └──┬─┘  └──┬─┘   └┬─┘  └────┬────┘  └────┬─────┘
│   │             │       │      │         │            name
│   │             │       │      │         last modified
│   │             │       │      size (bytes)
│   │             │       group
│   │             owner
│   permissions (read/write/execute for owner, group, others)
type: d = directory, - = file
```

(Permissions are covered in detail in a later class.)

---

## 7. Clear screen & history

| Command | What it does |
|---|---|
| `clear` or **Ctrl + L** | Clears the terminal screen |
| `history` | Lists the commands you typed earlier, numbered |
| `!25` | Re-run command number 25 from history |
| **↑ / ↓ arrow keys** | Scroll through previous commands |
| **Tab** | Auto-complete file and folder names (use it all the time!) |

---

## 8. Level 2 – Practice drill (do this on your instance)

```bash
whoami                 # who am I?
pwd                    # where am I?
ls -la                 # what's here (including hidden)?
cd /                   # go to the top
ls                     # see the main folders
cd /var/log            # go to the log folder
ll                     # long list of log files
cd ..                  # up one level → /var
cd -                   # back to /var/log
cd ~                   # home
sudo -i                # become root – notice $ → #
pwd                    # → /root
cd /home               # see all users' home folders
ls
exit                   # back to normal user
history                # see everything you did
clear
```

---

## 9. Level 3 – Architect view

- **Least privilege:** work as a normal user and use `sudo` only for the command that needs it. Running everything as root makes mistakes more damaging (`rm` as root can delete the whole system).
- **Know where things live** when troubleshooting EC2:
  - `/var/log/` – system and app logs (e.g. `/var/log/cloud-init-output.log` shows whether your **user data** script ran)
  - `/etc/` – configuration (e.g. web server config)
  - `/home/<user>/.ssh/authorized_keys` – the **public key** from your key pair lives here; this is how SSH login works (Day 8)
- **Auditing:** `history` is per-user and easy to erase. In companies, use **Session Manager** logging or **CloudTrail** for a reliable record of who did what.
- **Automation mindset:** every command you type by hand today can go into a **user data script**, a **custom AMI** or **Systems Manager Run Command**, so you can repeat it on 100 servers at once.

> 🎯 **Exam / interview tips:**
> - `$` = normal user, `#` = root.
> - `/` = root directory, `/root` = root user's home, `~` = current user's home.
> - Hidden files start with `.` and need `ls -a` to see.

---

## 10. Key terms

| Term | Meaning |
|---|---|
| Distribution (distro) | A flavour of Linux: Amazon Linux, Ubuntu, RHEL… |
| Prompt | The `user@host:dir$` text before you type |
| root | Linux super-user with full power |
| sudo | Run a command with root power |
| `/` | Root directory, top of the file system |
| `~` | Current user's home directory |
| `/root` | Root user's home directory |
| pwd | Show present working directory |
| cd | Change directory |
| ls / ll | List directory contents / long list |
| Hidden file | Name starts with `.`; shown with `ls -a` |
| Absolute path | Starts with `/` |
| Relative path | Starts from the current directory |

---

## 11. Quick quiz

1. In the prompt `ubuntu@ip-172-31-11-229:~$`, what are `ubuntu`, `ip-172-31-11-229`, `~` and `$`?
2. What does `#` at the end of the prompt mean?
3. Which command makes you root with a fresh login?
4. How do you go back to the previous user?
5. What is the difference between `/` and `/root`?
6. What does `~` mean when you're logged in as `ubuntu`?
7. Which command shows where you are right now?
8. You're in `/var/log`. Where does `cd ..` take you?
9. Which command shows hidden files?
10. `ll` gives "command not found". What do you use instead?

<details>
<summary>Show answers</summary>

1. Username; hostname (from the private IP); current directory (home); standard user
2. You are the root user
3. `sudo su -` or `sudo -i`
4. `exit` or Ctrl + D
5. `/` is the root directory (top of the file system); `/root` is the root user's home folder
6. `/home/ubuntu`
7. `pwd`
8. `/var`
9. `ls -a` (or `ls -la`)
10. `ls -l` (or `ls -la`)

</details>

---

## One-line summary

> **In Linux, `$` is a normal user and `#` is root (reach it with `sudo -i`); use `pwd` to see where you are, `cd` to move, `ls`/`ll` to list, and remember: `/` is the top of the tree, `~` is your home.**
