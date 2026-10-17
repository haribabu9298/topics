# Linux for Absolute Beginners

Welcome. This guide teaches you Linux from zero knowledge. No jargon without explanation. No commands without context.

By the end, you will understand what Linux is, how to navigate it, and how to get things done on it.

---

## Table of Contents

**Foundations (read these first)**
1. [What is Linux?](#1-what-is-linux)
2. [The Command Line — Your Tool](#2-the-command-line--your-tool)
3. [The Filesystem — One Big Tree](#3-the-filesystem--one-big-tree)
4. [Files and Folders — Navigation](#4-files-and-folders--navigation)
5. [Permissions — Who Can Do What](#5-permissions--who-can-do-what)
6. [Users and Groups](#8-users-and-groups)

**Core Skills**
7. [Running Programs](#6-running-programs)
8. [Text Files and Viewing Them](#7-text-files-and-viewing-them)
9. [Getting Information About Your System](#9-getting-information-about-your-system)
10. [Fixing Things When They Break](#10-fixing-things-when-they-break)

**Advanced Topics (explained simply)**
11. [Boot Sequence — How Linux Starts](#11-boot-sequence--how-linux-starts)
12. [systemd — The Manager of Everything](#12-systemd--the-manager-of-everything)
13. [Storage Beyond The Basics — LVM & RAID](#13-storage-beyond-the-basics--lvm--raid)
14. [Networking — Connecting Machines](#14-networking--connecting-machines)
15. [Package Managers — Installing Software](#15-package-managers--installing-software)
16. [Security Layers — SELinux and PAM](#16-security-layers--selinux-and-pam)
17. [Containers — Isolated Programs](#17-containers--isolated-programs)
18. [Performance — When Things Get Slow](#18-performance--when-things-get-slow)
19. [Backup and High Availability](#19-backup-and-high-availability)
20. [Interview Preparation](#20-interview-preparation)

---

## 1. What is Linux?

### The Basic Picture

Think of a computer in three layers:

```
┌─────────────────────────────────┐
│  YOU (typing commands)          │
├─────────────────────────────────┤
│  PROGRAMS (apps you run)        │  ← bash, Firefox, Word, etc.
├─────────────────────────────────┤
│  LINUX KERNEL (the core)        │  ← talks to hardware
├─────────────────────────────────┤
│  HARDWARE (CPU, disk, RAM)      │
└─────────────────────────────────┘
```

**Linux** is just the middle layer — the software that sits between your programs and the actual hardware. It is the referee.

- **Your programs** cannot talk to the disk directly. They ask Linux.
- **Your programs** cannot grab all the RAM. Linux parcels it out.
- **Your programs** cannot crash and take the whole machine down. Linux isolates them.

### What You Interact With

You do not interact with "Linux" directly. You interact with the **shell** — a program that reads what you type and runs other programs. The most common shell is called `bash`.

So when you see `$` at the start of a line in this guide, that means "type this into bash."

```bash
$ echo hello
hello
```

You typed `echo hello`. Bash saw this and ran a program called `echo` with the instruction "print hello". That program printed `hello` and exited.

### Linux vs "A Linux"

People use "Linux" to mean three different things:

| What they say | What they mean |
|---|---|
| "I use Linux" | I run a **Linux-based operating system** — Ubuntu, Debian, Red Hat, etc. |
| "The Linux kernel" | The core software (just the middle layer above). |
| "Linux desktop" | A computer using a Linux-based OS with a graphical interface. |

For this guide, "Linux" means **the operating system and everything that comes with it** — the kernel plus the programs, the filesystem, the permission system, all of it.

---

## 2. The Command Line — Your Tool

### The Shell Prompt

When you open a terminal, you see:

```
user@machine:~$
```

This is the **prompt**. It is waiting for you to type something. The `$` means you are a regular user (not the all-powerful `root` user). If you see `#`, you have superuser powers.

### Basic Command Structure

Every command follows a pattern:

```
command argument1 argument2
```

For example:

```bash
$ ls /home
```

- `ls` — the command (list files)
- `/home` — the argument (list files *in* this folder)

### Getting Help

If you forget what a command does:

```bash
$ ls --help                    # quick reminder
$ man ls                       # full manual (q to quit)
$ apropos "list files"         # find a command by what it does
```

### The Golden Rules

1. **Everything is case-sensitive.** `File.txt` and `file.txt` are different.
2. **Spaces matter.** `ls /home` is different from `ls/home`.
3. **Type carefully.** Mistyped commands just fail; they do not guess.

---

## 3. The Filesystem — One Big Tree

### Windows vs Linux

On Windows, you have separate drives: `C:\`, `D:\`, etc. Each one is its own world.

On Linux, **there is one tree that starts at `/` (the root)**. All drives, all folders, all files are branches of this single tree.

### The Top-Level Folders

When you first boot, you see:

```
/                           ← the root of everything
├── home/                   ← where regular users' files live
├── root/                   ← where the superuser (root) lives
├── var/                    ← files that change a lot (logs, temp data)
├── etc/                    ← system configuration (text files you can read and edit)
├── tmp/                    ← temporary files (everyone can write here, gets cleaned)
├── bin/                    ← programs you run
├── lib/                    ← libraries (code that programs use)
├── dev/                    ← devices (your disk, USB stick, etc.)
└── proc/                   ← live information about the system (we'll see this later)
```

Do not worry about the others for now. The important thing: **there is no `C:\` or `D:\` — it is all one tree starting at `/`.**

### Absolute vs Relative Paths

Every file has an **absolute path** — its full address from the root:

```
/home/alice/Documents/shopping.txt
```

This says: "start at root `/`, go into `home`, then `alice`, then `Documents`, and find `shopping.txt`."

You can also describe a location **relative to where you are right now**:

```bash
$ pwd                           # "print working directory" — where am I?
/home/alice
$ ls Documents                  # list files in Documents (relative)
$ ls /home/alice/Documents      # same thing, but absolute path
```

**Rules for paths:**
- `/something` — absolute (starts at root)
- `something` — relative (from where you are)
- `.` — current folder
- `..` — one level up

---

## 4. Files and Folders — Navigation

### The Essential Commands

```bash
pwd                     # where am I right now?
ls                      # list what is here
ls -l                   # list with details (long format)
ls -lh                  # list with human-readable file sizes
cd /path/to/folder      # go to a folder
cd ..                   # go up one level
cd ~                    # go to your home folder
mkdir foldername        # create a new folder
cp file1 file2          # copy a file
mv file1 file2          # move (or rename) a file
rm file                 # delete a file
rm -r folder            # delete a folder and everything in it
```

### Examples in Action

```bash
$ pwd
/home/alice

$ ls
Documents  Downloads  Pictures  Music

$ cd Documents
$ pwd
/home/alice/Documents

$ ls -lh
total 24K
-rw-r--r-- 1 alice alice  4.2K Jan 10 09:15 shopping.txt
-rw-r--r-- 1 alice alice   18K Jan  9 14:32 budget.xlsx
drwxr-xr-x 2 alice alice 4.0K Jan  8 11:02 Work

$ mkdir Projects
$ ls
shopping.txt  budget.xlsx  Work  Projects

$ cp shopping.txt shopping_backup.txt
$ ls
shopping.txt  shopping_backup.txt  budget.xlsx  Work  Projects
```

### A Dangerous Command

```bash
rm -r /
```

**Do not ever type this.** It deletes *everything* starting from the root. It will destroy your Linux installation.

The lesson: `rm` is **permanent**. There is no trash can, no undo. Be careful.

---

## 5. Permissions — Who Can Do What

### The Permission String

When you type `ls -l`, you see something like:

```
-rw-r--r--  1  alice  alice   1024  Jan 10 09:15  shopping.txt
```

The first part, `-rw-r--r--`, is the permission string. It answers: *"who is allowed to do what with this file?"*

### Breaking It Down

```
- rw- r-- r--
│ │   │   │
│ │   │   └─── everyone else can do this
│ │   └─────── people in the 'alice' group can do this
│ └─────────── the owner (alice) can do this
└───────────── file type: - = file, d = folder
```

The letters mean:

- `r` — read (look at the contents)
- `w` — write (change the contents)
- `x` — execute (run it as a program, or for folders: enter it)

### Examples

```
-rw-r--r--    owner can read and write, group can read, others can read
-rw-------    owner can read and write, nobody else can do anything
drwxr-xr-x    folder: owner full control, group can enter, others can enter
-rwxr-xr-x    a program: everyone can run it
```

### The Owner and Group

Every file has:

- An **owner** — usually the user who created it
- A **group** — a collection of users (we will skip groups for now)

When you try to read a file, Linux checks:

```
Is this YOUR file?           → use the owner rules
No. Are you in its GROUP?    → use the group rules
No. You are everyone else    → use the "other" rules
```

### Changing Permissions

```bash
chmod 644 shopping.txt          # owner: read+write, others: read only
chmod 755 script.sh             # owner: full, others: read and run
chmod 700 secrets               # owner only — nobody else can do anything
```

The numbers:

- `7` = `r+w+x` (read + write + run)
- `6` = `r+w` (read + write)
- `5` = `r+x` (read + run)
- `4` = `r` (read only)
- `0` = nothing

### The Superuser — `root`

There is one special user called `root` who bypasses all permissions. When you are `root`, you can read any file, write to any file, run any program, and do anything to anyone else's files.

```bash
$ whoami
alice
$ sudo whoami
root
```

`sudo` means "run this command as root." Linux will ask for your password first (the admin password, not necessarily your user password).

⚠️ **Do not use `sudo` casually.** It is powerful. Mistakes with `sudo` can break the whole system.

---

## 6. Running Programs

### Finding a Program

Programs live in folders like `/bin` and `/usr/bin`. When you type a command, bash searches these folders to find it.

```bash
$ which ls                      # where is the ls program?
/usr/bin/ls

$ which python
/usr/bin/python3
```

### Running a Program With Arguments

Most programs take **arguments** — extra information you give them:

```bash
$ grep error /var/log/app.log       # search for "error" in a log file
$ python3 script.py --verbose       # run a script with the --verbose option
```

### Pipes — Chaining Programs Together

One of Linux's superpowers is **pipes** — you can feed the output of one program into another:

```bash
$ cat /var/log/app.log | grep error
```

This means:

1. `cat` prints the file
2. The `|` pipe sends that output to `grep`
3. `grep` filters it to only lines containing "error"

Think of it like a factory assembly line: one worker's output becomes the next worker's input.

### Running Programs in the Background

If a program takes a long time, you can start it and get your prompt back:

```bash
$ python3 backup.py &              # the & means "run in the background"
[1] 12345                           # it tells you the process ID
$ ls                                # you can type more commands
$ fg                                # bring it back to the foreground
```

---

## 7. Text Files and Viewing Them

### Viewing a File

The most common task is reading files. Linux has several ways:

```bash
$ cat file.txt                      # print the whole file
$ less file.txt                     # page through it (press q to quit, / to search)
$ head -20 file.txt                # first 20 lines
$ tail -20 file.txt                # last 20 lines
```

For files you check often (like logs), `tail -f` is your friend:

```bash
$ tail -f /var/log/app.log         # watch the log live as it updates
# press Ctrl+C to stop watching
```

### Editing a File

Use a simple editor like `nano`:

```bash
$ nano file.txt
```

It shows you the commands at the bottom. Press `Ctrl+O` to save, `Ctrl+X` to exit.

Or use `vi` (more powerful but confusing at first):

```bash
$ vi file.txt
```

Press `i` to start editing, then press `Escape` and type `:wq` to save and exit.

### Searching Text

```bash
$ grep "error" file.txt                 # find lines with "error"
$ grep -i "ERROR" file.txt              # case-insensitive search
$ grep -r "error" /var/log/             # search inside many files
```

### Creating a File

```bash
$ echo "hello" > file.txt               # create (or overwrite) a file
$ echo "world" >> file.txt              # append to the end
```

---

## 8. Users and Groups

### What is a User?

A **user** is an identity. Each person who logs in is a different user. Each user has:

- A **username** (like `alice`)
- A **UID** — a unique number the system actually uses
- A **home folder** (like `/home/alice`)
- A **password** (ideally a strong one)

### The Root User

There is one special user, `root`, with UID 0. Root can do anything — read any file, change any file, run any command, do anything to anyone else's stuff.

On your personal computer, you might be root all the time. On a server, regular users are forbidden from being root and have to ask (via `sudo`) when they need permission.

### Becoming Root

```bash
$ sudo whoami                       # run one command as root
root
$ sudo -s                           # open a root shell (be careful!)
# whoami
root
# exit
$ whoami
alice
```

### Groups

A **group** is a collection of users. You use groups to share files:

```bash
$ groups alice                      # what groups does alice belong to?
alice wheel developers

$ ls -l
-rw-r-----  1  alice  developers  1024  file.txt

$ ls -l file.txt
-rw-r-----  1  alice  developers  1024  file.txt
```

This file is owned by `alice` and the group `developers`. Anyone in the `developers` group can read it (but not write to it).

### Creating Users (Requires Root)

```bash
$ sudo useradd -m newuser           # create a new user
$ sudo passwd newuser               # set their password
$ sudo userdel newuser              # delete a user
```

---

## 9. Getting Information About Your System

### What is the Machine Running?

```bash
$ uname -a                          # operating system details
$ lsb_release -a                    # Linux distribution info

$ hostnamectl                       # hostname and OS
$ whoami                            # current user
$ pwd                               # current folder
$ date                              # current time
```

### How Much Disk Space?

```bash
$ df -h                             # disk space (h = human-readable)

Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   45G   55G  45% /
/dev/sda2       200G  180G   20G  90% /home
```

- `/dev/sda1` is the main disk, 45GB used out of 100GB
- `/dev/sda2` is the home folder disk, 180GB used out of 200GB — getting full!

```bash
$ du -sh /home                      # how big is the /home folder?
87G     /home

$ du -sh /home/*                    # break it down by user
54G     /home/alice
33G     /home/bob
```

### How Much Memory?

```bash
$ free -h                           # RAM usage

               total  used  free  shared  buff/cache  available
Mem:            16Gi  8.2Gi 2.1Gi   512Mi     5.7Gi     7.5Gi
Swap:           4.0Gi  256Mi 3.7Gi
```

- `total` — how much RAM total
- `used` — how much programs are using right now
- `free` — completely unused (not as important as it looks!)
- `available` — how much is available for a new program to use
- `Swap` — slower disk space used if RAM fills up

**The key thing:** `free` being low is normal and not a problem. What matters is `available`.

### What Programs Are Running?

```bash
$ ps aux                            # list all running programs

USER    PID     %CPU %MEM    COMMAND
root    1       0.1  0.1     /sbin/init
alice   5821    2.3  4.5     /usr/bin/python3 backup.py
bob     6102    0.0  0.2     bash
```

- `PID` — the program's ID number
- `%CPU` — how much CPU it is using (100% = one full core)
- `%MEM` — how much RAM it is using
- `COMMAND` — what is running

### Is a Program Running?

```bash
$ ps aux | grep nginx               # search for nginx
```

### Logs — What Happened?

Most programs write logs to `/var/log/`:

```bash
$ ls /var/log/
auth.log  syslog  apache2/  nginx/  mysql/  ...

$ tail -f /var/log/syslog          # watch the system log live
```

---

## 10. Fixing Things When They Break

### The Mindset

When something breaks, do not panic. Follow this order:

1. **What is the problem?** — get the exact error message
2. **Where is it happening?** — find the relevant log
3. **Why is it happening?** — read the log carefully
4. **How do I fix it?** — once you understand the cause, the fix is usually obvious

Most of the time, you are just **reading error messages carefully**. They usually say exactly what went wrong.

### Classic Problems and Fixes

#### "Permission denied"

```bash
$ cat /etc/shadow
cat: /etc/shadow: Permission denied
```

This means you do not have permission. Check who owns the file:

```bash
$ ls -l /etc/shadow
-rw-r----- 1 root shadow 1024 /etc/shadow
```

The owner is `root`, the group is `shadow`, and you are neither. Solutions:

```bash
$ sudo cat /etc/shadow              # run as root
$ sudo usermod -aG shadow alice     # add yourself to the shadow group (then log out and back in)
```

#### "No such file or directory"

```bash
$ cat /home/alice/shopping.txt
cat: /home/alice/shopping.txt: No such file or directory
```

The file does not exist, or you typed the path wrong. Check:

```bash
$ ls -l /home/alice/ | grep shopping
$ pwd                               # are you in the right place?
```

#### A Program Will Not Start

```bash
$ python3 script.py
python3: command not found
```

The program is not installed, or not in your PATH. Check:

```bash
$ which python3                     # where is python?
$ python3 --version                # is it installed?
```

If not installed:

```bash
$ sudo apt install python3          # on Ubuntu/Debian
$ sudo dnf install python3          # on Red Hat/Fedora
```

#### The Disk is Full

```bash
$ df -h
/dev/sda1       100G  100G    0B 100% /
```

Everything is 100% used. You need to delete something:

```bash
$ du -sh /*                         # which folder is biggest?
$ rm old_file.txt                   # delete old files
$ sudo rm -rf /tmp/*                # empty the temp folder
```

#### A Program is Stuck

```bash
$ ps aux | grep stuck_program
alice   12345   95.0   50.0   stuck_program

$ kill 12345                        # send a stop signal
$ ps aux | grep 12345               # is it gone?
alice   12345   95.0   50.0   stuck_program

$ kill -9 12345                     # force kill
```

The numbers mean:

- `kill <PID>` — ask nicely (the program can ignore this)
- `kill -9 <PID>` — force kill (cannot be ignored)

### Getting Help

When you are stuck, the error message is your best friend. Copy the exact message and:

1. Read it carefully — it usually says what is wrong
2. Search online (paste the message)
3. Check the manual: `man command`
4. Ask someone, but paste the error message

---

## Summary — The Essentials

| Task | Command |
|---|---|
| See where I am | `pwd` |
| List files | `ls -lh` |
| Move around | `cd /path` |
| Create a folder | `mkdir name` |
| View a file | `cat file` or `less file` |
| Edit a file | `nano file` |
| Search text | `grep word file` |
| Run as root | `sudo command` |
| Check disk space | `df -h` |
| Check RAM | `free -h` |
| See running programs | `ps aux` |
| Watch a log | `tail -f /var/log/syslog` |
| Permissions | `chmod 755 file` |
| Copy a file | `cp file1 file2` |
| Delete a file | `rm file` |
| Get help | `man command` or `command --help` |

---

## Keyboard Shortcuts

These work in bash and will save you time:

| Shortcut | What it does |
|---|---|
| `Ctrl+C` | Stop the current command |
| `Ctrl+Z` | Pause the current command (type `fg` to resume) |
| `Ctrl+L` | Clear the screen |
| `Ctrl+A` | Jump to the beginning of the line |
| `Ctrl+E` | Jump to the end of the line |
| `Ctrl+R` | Search through your command history |
| `Tab` | Auto-complete (type part of a command and press Tab) |
| `↑` `↓` | Navigate through recent commands |

---

## 11. Boot Sequence — How Linux Starts

### What Happens When You Press Power

When you turn on a computer, it cannot run Linux directly. It has to **bootstrap itself** — build itself up from nothing, step by step. Think of it like waking up: you cannot go from totally asleep to running a marathon. You have to sit up, get dressed, have breakfast, then run.

Here is the sequence:

```
1. POWER ON
   ↓
2. FIRMWARE (BIOS or UEFI) — very basic code built into the motherboard
   → tells the computer "here is your hardware"
   → finds the hard disk
   ↓
3. BOOTLOADER (GRUB) — a small program on the disk
   → shows you a menu (if you have multiple operating systems)
   → loads the Linux kernel into memory
   ↓
4. LINUX KERNEL wakes up
   → detects all hardware (CPU, RAM, disks, network)
   → loads drivers to talk to the hardware
   ↓
5. initramfs (a tiny temporary filesystem)
   → is just enough Linux to find the real root filesystem
   → mounts the main disk
   ↓
6. SYSTEMD (PID 1 — the first program)
   → starts everything else (services, daemons, your desktop)
   ↓
7. READY — you see a login prompt
```

### Why This Matters

Each stage can fail, and the failure looks different:

- **BIOS fails** — the computer beeps and does nothing (you see no message)
- **GRUB fails** — you see a black screen with "GRUB>"
- **Kernel fails** — you see "Kernel panic" message
- **initramfs fails** — you see "Cannot mount root filesystem"
- **systemd fails** — you boot to a black screen (emergency mode) and cannot log in

### Recovery Mode

If the machine is broken, you can often press a key at boot to get a recovery menu:

- Press **Spacebar** at the GRUB menu to pause it
- Press **`e`** to edit and boot with special options
- Press **`rd.break`** to break into the initramfs for emergency access

This is why Linux admins laugh when you say a system is "completely broken" — you can almost always get back in.

---

## 12. systemd — The Manager of Everything

### What Is systemd?

When the kernel finishes starting up, it runs a program called `systemd`. This program's only job is: **start and manage everything else on the machine**.

Think of systemd as a **restaurant manager opening up**. There are a million things to do:

- Start the ovens
- Unlock the doors
- Turn on the lights
- Call the staff
- Check that tables are set up

Some things depend on others (you cannot seat customers before opening the doors). A good manager works out the dependencies and starts everything in parallel where possible.

### Services

A **service** is a program that runs continuously in the background. Examples:

- `nginx` — web server
- `mysql` — database
- `sshd` — lets you log in remotely
- `docker` — container runtime

You control services with `systemctl`:

```bash
$ systemctl start nginx              # start it right now
$ systemctl stop nginx               # stop it right now
$ systemctl restart nginx            # stop, then start
$ systemctl status nginx             # is it running?
$ systemctl enable nginx             # start it at boot time
$ systemctl disable nginx            # do not start it at boot
```

### Enabled vs Running (This Confuses Beginners!)

- **Running** = it is working right now
- **Enabled** = it will start automatically at boot

These are **independent**:

| State | Running? | At Boot? | Meaning |
|---|---|---|---|
| Started, not enabled | Yes | No | Working now, but will not come back after reboot |
| Enabled, not running | No | Yes | Will start at boot, but not running now |
| Enabled and running | Yes | Yes | Working and will start at boot |
| Neither | No | No | Not running, will not start at boot |

```bash
$ systemctl start nginx              # running now
$ systemctl enable nginx             # will start at boot
$ systemctl enable --now nginx       # does both at once (recommended)
```

### Checking If a Service Is OK

```bash
$ systemctl status nginx

● nginx.service - The NGINX HTTP and Reverse Proxy Server
   Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2025-01-10 09:15:22 UTC; 2h 30min ago
```

Read this line by line:

- `Loaded: loaded` — the config file was found and read
- `enabled` — it is set to start at boot
- `Active: active (running)` — it is working right now
- `2h 30min ago` — it has been running for 2.5 hours

If something is wrong:

```bash
$ systemctl status nginx

● nginx.service - The NGINX HTTP and Reverse Proxy Server
   Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Mon 2025-01-10 10:22:11 UTC; 5min ago
```

Read the logs:

```bash
$ journalctl -u nginx --no-pager -n 50       # last 50 lines of nginx logs
$ journalctl -u nginx -f                      # watch logs live
```

### Targets — Milestones

A **target** is a milestone that says "the system is in a certain state." Common ones:

- `multi-user.target` — text login (no graphical desktop), all services running
- `graphical.target` — graphical desktop loaded
- `rescue.target` — minimal environment for fixing problems
- `emergency.target` — absolute minimum (one shell, nothing else)

```bash
$ systemctl get-default              # which target do we boot to?
graphical.target

$ systemctl set-default multi-user.target    # change it (reboot to take effect)
$ systemctl isolate rescue.target    # jump to a different target right now (dangerous!)
```

### Custom Services

You can write your own `.service` file to make systemd run your program:

```bash
$ sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Custom Application
After=network.target

[Service]
ExecStart=/opt/myapp/run.sh
Restart=always
User=myapp

[Install]
WantedBy=multi-user.target
```

Then:

```bash
$ sudo systemctl daemon-reload       # re-read the config files
$ sudo systemctl enable myapp        # start at boot
$ sudo systemctl start myapp         # start now
$ systemctl status myapp
```

---

## 13. Storage Beyond The Basics — LVM & RAID

### The Problem

Normally when you create a filesystem, you carve it out of a single disk partition. But what if:

- You guess the size wrong and run out of space?
- You want a filesystem to span multiple disks?
- You want to take a snapshot for backup?

**LVM** fixes these problems.

### LVM — Logical Volume Manager

LVM adds a **layer** between your hard drive and your filesystem. Instead of filesystems sitting directly on disk partitions, they sit on "logical volumes" that are built from a pool of disk space.

**The water tank analogy:**

```
Physical Disks (/dev/sda, /dev/sdb)
        ↓ (pour into)
    Tank (Volume Group "vg_data")
        ↓ (draw from)
    Glasses (Logical Volumes "lv_app", "lv_logs")
        ↓ (fill with)
    Filesystem (ext4 / XFS)
```

**Why this is useful:**

1. **Grow a filesystem while it is running** — add more disk space to the tank, make a glass bigger, all without downtime
2. **Span multiple disks** — one logical volume can pull space from multiple physical disks
3. **Shrink filesystems** — take back space you do not need

**Example: Add a second disk to a machine**

```bash
$ sudo fdisk /dev/sdb                # create a partition on the new disk
$ sudo pvcreate /dev/sdb1            # "pour" it into the tank (Physical Volume)
$ sudo vgextend vg_data /dev/sdb1    # add it to the existing tank (Volume Group)
$ sudo lvextend -L +50G vg_data/lv_app    # give 50GB more to the glass
$ sudo xfs_growfs /app               # tell the filesystem it has more room
```

After this, `/app` is now 50GB larger, with zero downtime.

### RAID — Redundant Disk Arrays

**The problem:** what if a disk dies? You lose all the data.

**RAID's answer:** have multiple copies of your data across different disks. If one dies, the others keep working.

**RAID 1 (mirroring):**

```
Disk A: [data]
Disk B: [copy of data]

If Disk A dies, Disk B keeps working.
If Disk B dies, Disk A keeps working.

Cost: you need 2 disks to store 1 disk's worth of data.
```

**RAID 5 (striping with parity):**

```
Disk A: [data block 1]
Disk B: [data block 2]
Disk C: [parity — can recreate either A or B]

If any one disk dies, the other two can recreate its data.
If Disk A dies, you rebuild it using Disk B and Disk C.

Cost: you need 3 disks to store 2 disks' worth of data.
```

**Example RAID setup:**

```bash
$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
$ sudo mkfs.ext4 /dev/md0
$ sudo mount /dev/md0 /data
```

If `/dev/sdb1` dies:

```bash
$ sudo mdadm --manage /dev/md0 --fail /dev/sdb1
$ sudo mdadm --manage /dev/md0 --remove /dev/sdb1
# Physically replace the disk
$ sudo mdadm --manage /dev/md0 --add /dev/sdb1
# It automatically rebuilds (slow, but automatic)
```

### When to Use Each

- **No RAID** — testing, sandboxes, temporary machines (data loss is okay)
- **RAID 1** — databases, small critical systems, when read speed matters
- **RAID 5** — large storage, good balance of safety and speed
- **RAID 6** — very large storage (RAID 5 rebuild is risky if multiple disks fail)

---

## 14. Networking — Connecting Machines

### The Basics

Every computer on a network has:

- An **IP address** — its postal address (like `192.168.1.42`)
- A **gateway** — the router that routes traffic to other networks
- A **DNS server** — translates names to addresses (google.com → 142.250.185.46)

### IP Addresses

An IP address is four numbers from 0-255, separated by dots:

```
192.168.1.42
│   │   │  │
│   │   │  └─ fourth octet (host ID)
│   │   └───── third octet
│   └───────── second octet
└───────────── first octet
```

**Subnetting** — a network is a range of addresses. A **/24 network** means the first 24 bits are the same:

```
192.168.1.0/24  means:
192.168.1.0
192.168.1.1
192.168.1.2
...
192.168.1.255   (256 addresses total)
```

### Checking Your Network

```bash
$ ip addr                           # show IP addresses
$ ip route                          # show routing table
$ ping 8.8.8.8                      # can you reach this address?
$ traceroute google.com             # what is the path to this address?
$ nslookup google.com               # what is the IP of this domain?
$ netstat -tuln                     # what ports are listening?
```

### Firewalls

A firewall blocks unwanted traffic. You configure it with rules:

```bash
$ sudo ufw enable                   # turn on the firewall
$ sudo ufw allow 22                 # allow SSH (port 22)
$ sudo ufw allow 80                 # allow HTTP (port 80)
$ sudo ufw deny 3306                # block MySQL (port 3306)
$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
3306                        DENY        Anywhere
```

### Ports

A **port** is a numbered "door" on your machine. Different services listen on different ports:

- 22 — SSH (remote login)
- 80 — HTTP (web)
- 443 — HTTPS (secure web)
- 3306 — MySQL (database)
- 5432 — PostgreSQL (database)

```bash
$ netstat -tulnp                    # show all listening ports and which program owns them
Proto  Local Address    State   Program
tcp    0.0.0.0:22       LISTEN  sshd
tcp    0.0.0.0:80       LISTEN  nginx
tcp    0.0.0.0:443      LISTEN  nginx
```

If you try to connect to port 80 and it fails, check:

```bash
$ netstat -tulnp | grep 80          # is anything listening on 80?
$ sudo ufw status | grep 80         # is the firewall blocking it?
$ sudo systemctl status nginx       # is the web server running?
```

---

## 15. Package Managers — Installing Software

### What is a Package?

Instead of downloading a program from a website and clicking "Install", Linux uses **package managers** — centralized stores of software that the system can install, update, and remove automatically.

A **package** is a bundle containing:

- The program itself
- Configuration files
- Dependencies (other programs it needs to work)
- Instructions for where to put things

### Two Main Package Managers

**On Ubuntu / Debian (and derivatives):**

```bash
$ sudo apt update                   # refresh the list of available packages
$ sudo apt install nginx            # install a package
$ sudo apt remove nginx             # uninstall
$ sudo apt upgrade                  # upgrade all installed packages
$ apt search python                 # search for packages with "python" in the name
$ apt show python3                  # show info about a package
```

**On Red Hat / Fedora / Rocky (and derivatives):**

```bash
$ sudo dnf update                   # refresh the list of available packages
$ sudo dnf install nginx            # install a package
$ sudo dnf remove nginx             # uninstall
$ sudo dnf upgrade                  # upgrade all installed packages
$ dnf search python                 # search for packages
$ dnf info python3                  # show info about a package
```

**Difference:** `apt` vs `dnf`. They do the same job, just different names and slightly different commands. Pick your Linux distribution and learn one.

### Repositories

Packages live on **repositories** — servers operated by the distribution. Ubuntu has one, Red Hat has one, etc.

Sometimes you need software from a *different* repository:

```bash
# Ubuntu example: add Docker's repository
$ sudo add-apt-repository ppa:docker/docker
$ sudo apt update
$ sudo apt install docker.io

# Red Hat example: add Docker's repository
$ sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
$ sudo dnf install docker-ce
```

### When Packages Break Your System

Sometimes an update breaks something. You can downgrade:

```bash
# Ubuntu
$ sudo apt install nginx=1.18.0-1    # install a specific version

# Red Hat
$ sudo dnf downgrade nginx
```

Or see what changed:

```bash
$ apt changelog nginx               # Ubuntu
```

---

## 16. Security Layers — SELinux and PAM

### The Two Layers

Linux has **two security layers**. They check permissions differently:

1. **Standard permissions** (what we covered earlier) — owner / group / other
2. **SELinux** — "is this program *allowed* to do this?"

Think of it like an office building:

- **Standard permissions** = the key to the room
- **SELinux** = your job title

You may have a key to the finance office (permission), but if your job is "cleaner," SELinux stops you from going in (policy).

### SELinux

SELinux rules are stricter than regular permissions. Every file and every program has a **label**:

```bash
$ ls -Z /etc/shadow
-rw-r----- root shadow system_u:object_r:shadow_t /etc/shadow
                                           └─ this is the label
```

Programs have labels too:

```bash
$ ps -eZ | grep nginx
system_u:system_r:httpd_t - nginx

nginx runs as httpd_t, and SELinux says "what can httpd_t do?"
```

If a program tries to do something its label does not allow, SELinux blocks it and logs it:

```bash
$ sudo tail -f /var/log/audit/audit.log | grep denied
```

### When SELinux Blocks You

The classic symptom:

```
Permissions look right (644, owned by correct user).
Program still gets "Permission denied".
```

Check SELinux:

```bash
$ getenforce
Enforcing

$ ausearch -m AVC -ts recent | tail
type=AVC msg=audit(...): avc: denied { read } for pid=1234 ...
```

Fix it:

```bash
$ restorecon -Rv /var/www           # reset labels to policy defaults
```

Or check if SELinux is the problem by temporarily disabling it:

```bash
$ sudo setenforce 0                 # turn off SELinux (temporary)
$ sudo setenforce 1                 # turn back on
```

To disable permanently:

```bash
$ sudo nano /etc/selinux/config
# Change: SELINUX=enforcing
# To:     SELINUX=disabled
# Then reboot
```

**Interview Tip:** Never say "I will just disable SELinux." Instead say: "I will check the audit log to see what is denied, then fix the policy."

### PAM — Pluggable Authentication Modules

PAM is the framework that handles **login and authentication**. When you type your password, PAM checks it against:

- Your password file
- LDAP (company directory)
- Kerberos (enterprise auth)
- A hardware token

You normally do not touch PAM — it just works. But if your company uses Active Directory or LDAP, PAM is what connects Linux to it.

Example: to require a password to be 12 characters minimum:

```bash
$ sudo nano /etc/security/pwquality.conf
minlen = 12

# Then users must have 12-character passwords
```

---

## 17. Containers — Isolated Programs

### What is a Container?

A container is a program running in **isolation**. It has:

- Its own filesystem (thinks `/` is a private directory)
- Its own set of processes (cannot see other containers' programs)
- Its own network (private IP address)
- Resource limits (can use at most X GB of RAM)

**But it runs on the same kernel as everything else** — unlike a virtual machine, there is no separate operating system.

### The Analogy

- **Virtual Machine** = a separate computer inside your computer (own CPU, own disk, own OS)
- **Container** = a box with your program, with walls between it and other boxes

### Docker

**Docker** is the most popular tool for running containers. A **Docker image** is like a recipe, and a **container** is the meal you cook from the recipe.

```bash
$ docker pull nginx                 # download the recipe
$ docker run -d -p 80:80 nginx      # cook a meal from it
  docker: running in background (-d)
          port 80 on me (-p) → port 80 in container
          recipe name: nginx

$ docker ps                         # what containers are running?
CONTAINER ID  IMAGE  PORTS
abc123        nginx  0.0.0.0:80->80/tcp

$ docker exec -it abc123 bash       # open a shell inside a container
# ls
bin boot dev etc ...
```

### Why Containers Matter

1. **Isolation** — if a container crashes, it does not affect others
2. **Portability** — a Docker image runs the same on your laptop and on production
3. **Easy to destroy** — you can start and stop containers instantly without affecting the base system
4. **Resource limits** — stop one program from eating all RAM

### Container Orchestration

When you have many containers (thousands), you need a tool to manage them. **Kubernetes** schedules containers across many machines:

- "Run 5 copies of nginx"
- "If one dies, start another"
- "Spread them across 3 different servers"
- "Automatically scale to 10 copies when load is high"

You do not need to understand Kubernetes as a beginner, but it is what runs most containers in production.

---

## 18. Performance — When Things Get Slow

### The Symptoms

Your machine is slow. **Where is the bottleneck?** It is one of four things:

| Bottleneck | Symptom | Check with | Fix by |
|---|---|---|---|
| **CPU** | High load, spinning | `top`, `htop` | Stop unnecessary programs, get faster CPU |
| **RAM** | Out of memory, swapping | `free -h`, `vmstat` | Add more RAM, identify memory leaks |
| **Disk** | I/O wait, disk red | `iostat -x`, `iotop` | Upgrade to SSD, reduce disk writes |
| **Network** | Cannot reach server, timeouts | `ping`, `traceroute` | Check connection, fix networking, get more bandwidth |

### Checking Each One

**CPU:**

```bash
$ top
Cpu(s): 45.2%us  5.3%sy  0.0%ni 49.1%id

If %id (idle) is high, CPU is not the problem.
If %id is low, CPU is maxed out.
```

**RAM:**

```bash
$ free -h
Mem:          available 8G    ← can a new program get 8G?
Swap:         16G used        ← if swap is being used, RAM is tight
```

**Disk:**

```bash
$ iostat -x 1 5
await: 50ms        ← time a disk read takes
                    < 5ms = fast
                    > 10ms = slow

If await is high, upgrade to SSD.
```

**Network:**

```bash
$ ping 8.8.8.8
round-trip min/avg/max = 10/15/20 ms

If ping fails or times out, network is down.
```

### Load Average

**Load average** = how many programs are waiting for CPU time.

```bash
$ uptime
load average: 2.5, 1.8, 1.2
            1min  5min  15min

On a 4-core machine:
- load 2.5 = comfortable (plenty of idle CPU)
- load 4.0 = balanced (CPU always busy)
- load 8.0 = overloaded (programs are waiting)
```

To see how many cores you have:

```bash
$ nproc
4

Rule: divide load by nproc.
Load 12 on 4-core = 3.0 = overloaded
Load 12 on 16-core = 0.75 = comfortable
```

### The Profiling Mindset

When something is slow:

1. **Reproduce the slowness** — can you make it happen again?
2. **Check the four bottlenecks** — which one is maxed?
3. **Narrow down to the program** — `top`, `ps`, `iotop` — what program is doing it?
4. **Get more data** — run `strace` to see what the program is doing
5. **Fix it** — optimize the program, add resources, or remove it

Example: MySQL is slow.

```bash
$ top
mysql is using 95% CPU

$ strace -p <mysql-pid>
% time     seconds  usecs/call     calls    system call
  50.0      10.0    10             1000     read
  30.0       6.0     6             1000     write

It is doing a lot of disk I/O. Check if you have an index.
Or if the disk is slow (iostat -x).
```

---

## 19. Backup and High Availability

### Why Backup Matters

"If it is not backed up, it will be lost." This is not pessimism — it is history. Disks fail. Ransomware happens. Accidents happen.

### The 3-2-1 Rule

A good backup strategy has:

- **3 copies** of your data (original + 2 backups)
- **2 different media** (disk + tape, or disk + cloud)
- **1 offsite** (at least one copy far away, in case the building burns down)

```
Laptop (1)
  ↓
External disk (2)
  ↓
Cloud backup (3) — in a different city
```

### Full vs Incremental

**Full backup** — copy everything.

```
Monday: backup 100GB    (take 2 hours)
Tuesday: backup 100GB   (take 2 hours)
Wednesday: backup 100GB (take 2 hours)
```

**Incremental backup** — copy only what changed.

```
Monday: backup 100GB (everything)        (2 hours)
Tuesday: backup 5GB (only what changed)  (10 minutes)
Wednesday: backup 3GB (only what changed) (5 minutes)
```

Incremental is faster. To restore, you need the last full + all incrementals since.

### High Availability

**Availability** = the percentage of time a service is working.

- 99% available = down 7 minutes per week
- 99.9% available = down 43 seconds per week

To achieve high availability, you use **redundancy**:

```
Load Balancer
  ↓
  ├─ Server 1 (database backup running)
  └─ Server 2 (database backup running)

If Server 1 dies, Server 2 takes over. Customers do not notice.
```

### Recovery Time Objective (RTO)

How long can you be down before it costs too much?

- A blog: 1 day is fine
- A bank: 1 hour is too much
- A hospital: 5 minutes is too much

This determines how much redundancy you need.

### Tools

- **Backup tools:** `rsync` (copy), `tar` (archive), `mysqldump` (database backup)
- **HA tools:** `keepalived` (shared IP), `heartbeat` (cluster), `etcd` (distributed config)

Simple example:

```bash
$ rsync -av /home/ backup@backup-server:/backups/home/
# Copy everything in /home/ to a backup server, mirror any deletions (-a)
```

---

## 20. Interview Preparation

If you are interviewing for a Linux role, these are the questions you will get. Remember: they want to hear *how* you think, not just the right answer.

### Core Concepts (Explain These Simply)

**Q: What is Linux?**

A: Linux is the kernel — the software between your programs and the hardware. It decides what each program can do. The distribution (Ubuntu, Red Hat) adds the tools and package manager.

**Q: How does a process start?**

A: A running program is a process. It starts when another process forks (clones itself) and then execs (replaces itself with a different program). PID 1 (systemd) started it, or it started from a shell.

**Q: What are permissions?**

A: Three sets of rules: owner, group, other. Each has read/write/execute. The system checks which one applies and uses that rule. Root (UID 0) bypasses all permissions.

**Q: What happens when the disk is full?**

A: Programs cannot write new files. Existing programs keep running but may start failing when they try to log or write data. You find the largest folder, delete old files, or resize the filesystem.

**Q: How do you find a slow process?**

A: Use `top` to see what is using CPU, `iostat -x` for disk, `free -h` for RAM, `netstat` for network. Look at the bottleneck, then narrow down to the offending program.

### Practical Scenarios (What You Would Actually Do)

**Q: A service is not starting. How do you debug?**

A:

1. Check status: `systemctl status service`
2. Read the error in the output
3. Check logs: `journalctl -u service --no-pager -n 50`
4. Fix the issue (bad config, missing dependency, wrong permissions)
5. Restart: `systemctl restart service`

**Q: You cannot log in via SSH. What do you check?**

A:

1. Is the SSH server running? `systemctl status sshd`
2. Is it listening on the right port? `netstat -tulnp | grep ssh`
3. Is the firewall blocking it? `ufw status`
4. Can you reach the server? `ping <ip>`
5. Check logs: `tail -f /var/log/auth.log` then try to connect again

**Q: The filesystem is read-only. Why?**

A:

1. Check if the disk has errors: `dmesg | grep error`
2. Remount with write permissions: `mount -o remount,rw /`
3. Run fsck to check for corruption: `fsck /dev/sda1` (when unmounted)

**Q: You need to add 100GB to a filesystem. What is the process?**

A:

1. Attach a new disk
2. Partition it: `fdisk /dev/sdb`
3. Add it to LVM: `pvcreate /dev/sdb1` then `vgextend vg_name /dev/sdb1`
4. Extend the logical volume: `lvextend -L +100G vg_name/lv_name`
5. Grow the filesystem: `xfs_growfs /mountpoint` or `resize2fs /dev/mapper/vg_name-lv_name`

**Q: A process is in a stuck in `D` state and `kill -9` does not work. Why?**

A: `D` state means it is waiting for disk I/O and cannot be interrupted. It is waiting for the kernel to finish a disk operation. Usually the disk is hung or NFS is disconnected. Force a reboot or fix the storage.

### What Interviewers Are Listening For

- **Can you explain it simply?** Jargon is okay if you explain it. Simple is always better than fancy.
- **Do you know what to check?** The right answer is not remembering a command, it is knowing *what* to look at.
- **Can you think through a problem?** Walk through your debugging process. Show that you have a system.
- **Do you know when to stop?** "I would ask the user what is most critical: speed or cost?" is a good answer.

### Red Flags (Do Not Say These)

- "I would just restart it" — shows you do not debug
- "I disabled SELinux" — shows you give up on security
- "I changed the root password by boot in single-user mode" — only if asked, otherwise keeps it secret
- "I would run as root all the time" — dangerous thinking
- "I have not used Linux in a while" — okay if true, but show you know the fundamentals

### Green Flags (Do Say These)

- "I would check the logs first"
- "I need to understand the problem before fixing it"
- "Can you tell me what changed recently?"
- "I would test this in a non-production environment first"
- "Here is what I would monitor going forward"

---

## Summary — The Full Picture

```
You turn on a computer
      ↓
Firmware → Bootloader (GRUB) → Kernel → initramfs → systemd
      ↓
systemd starts services
      ↓
Services run in containers or on bare metal
      ↓
Permissions and SELinux control access
      ↓
If it breaks: check logs, check status, diagnose with tools
      ↓
Back up your data, keep it running, and sleep well
```

The fundamentals you learned:

1. **Linux** = kernel + everything else
2. **Permissions** = owner/group/other + root bypass
3. **Processes** = fork and exec
4. **Files** = one tree from `/`
5. **Services** = programs that keep running
6. **Backup** = 3-2-1 rule
7. **Debugging** = find the bottleneck, read the logs, think systematically

Master these seven, and you can learn any Linux distribution.

---

## Next Steps

Now that you understand the basics:

1. **Open a terminal** and navigate to a few folders using `cd` and `ls`
2. **Create a folder** with `mkdir` and files with `echo "text" > file`
3. **Change permissions** with `chmod` and see how it affects what you can do
4. **Read a log file** with `less /var/log/syslog` and search with `/error`
5. **Check your disk** with `df -h` and see which folders are biggest with `du -sh /`

The best way to learn Linux is by doing. Break things in a safe environment (a virtual machine), read the error messages, and fix them. That is how you become comfortable with it.

Good luck!
