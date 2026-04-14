# 01 — Linux File Hierarchy & Navigation
---

## WHAT IS THE LINUX FILE HIERARCHY?
Linux organises everything in one big tree starting from **/** (root).
Everything — files, folders, devices, processes — lives somewhere in this tree.

```
/
├── bin/
├── boot/
├── dev/
├── etc/
├── home/
├── lib/
├── media/
├── mnt/
├── opt/
├── proc/
├── root/
├── run/
├── sbin/
├── srv/
├── sys/
├── tmp/
├── usr/
└── var/
```

---

## EVERY DIRECTORY EXPLAINED

| Directory | Full Name | What it contains |
|---|---|---|
| `/` | Root | Top of everything — start of the whole tree |
| `/bin` | Binaries | Essential commands — ls, cat, cp, mv, ping |
| `/boot` | Boot | Bootloader and kernel files — do not touch |
| `/dev` | Devices | Hardware as files — /dev/sda (hard disk), /dev/null |
| `/etc` | Et Cetera | All system config files — passwords, network, services |
| `/home` | Home | Personal folders for each user — /home/abdu |
| `/lib` | Libraries | Shared code used by programs in /bin and /sbin |
| `/media` | Media | Auto-mounted drives — USB, CD |
| `/mnt` | Mount | Manually mounted drives and filesystems |
| `/opt` | Optional | Third party software you install manually |
| `/proc` | Process | Live system info — CPU, memory, running processes |
| `/root` | Root home | Home folder for the root user only |
| `/run` | Runtime | Temporary files for running processes |
| `/sbin` | System Binaries | Admin commands — fdisk, iptables, reboot |
| `/srv` | Service | Data for services like web server, FTP |
| `/sys` | System | Kernel and hardware info |
| `/tmp` | Temporary | Temporary files — cleared on reboot |
| `/usr` | User | Programs, libraries, docs for all users |
| `/var` | Variable | Logs, databases, mail, print queues |

---

## MOST IMPORTANT DIRECTORIES — RED TEAM

```
/etc/passwd        → list of all users on system
/etc/shadow        → hashed passwords (root only)
/etc/hosts         → local DNS — hostname to IP mapping
/etc/crontab       → scheduled tasks — privesc target
/etc/sudoers       → who can run sudo — privesc target
/tmp               → world writable — good for dropping files
/var/log           → all system logs
/home/username/    → user files, SSH keys, history
/root/             → root user files (need root to read)
/proc/self/environ → current process environment variables
```

---

## NAVIGATION COMMANDS — BEGINNER

```bash
# Where am I right now?
pwd

# List files in current directory
ls

# List with details (permissions, size, owner)
ls -l

# List including hidden files (start with .)
ls -a

# List with details AND hidden files
ls -la

# Change directory
cd /etc
cd /home/abdu
cd ..          # go up one level
cd ~           # go to your home directory
cd -           # go back to previous directory

# Clear the screen
clear
```

---

## FILE OPERATIONS — BEGINNER

```bash
# Create a file
touch filename.txt

# Create a directory
mkdir myfolder
mkdir -p folder/subfolder/another   # create nested folders

# Copy a file
cp file.txt /tmp/file.txt

# Copy a directory
cp -r myfolder/ /tmp/myfolder/

# Move or rename
mv file.txt newname.txt             # rename
mv file.txt /tmp/file.txt           # move

# Delete a file
rm file.txt

# Delete a directory and everything inside
rm -rf myfolder/

# Read a file
cat file.txt

# Read large file page by page
less file.txt                       # q to quit
more file.txt

# Read first 10 lines
head file.txt
head -n 20 file.txt                 # first 20 lines

# Read last 10 lines
tail file.txt
tail -n 20 file.txt                 # last 20 lines
tail -f /var/log/syslog             # follow live updates
```

---

## FINDING FILES — INTERMEDIATE

```bash
# Find a file by name
find / -name "passwd"
find /etc -name "*.conf"
find /home -name "*.txt"

# Find files by permission (SUID — red team important)
find / -perm -4000 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Find writable directories
find / -writable -type d 2>/dev/null

# Find files owned by root
find / -user root -type f 2>/dev/null

# Locate a file (faster than find, uses database)
locate passwd
updatedb                            # update the locate database
```

---

## FILE PERMISSIONS — IMPORTANT

Every file has 3 permission sets: **owner**, **group**, **others**

```
-rwxr-xr--
│└─┬─┘└─┬─┘└─┬─┘
│  │    │    └── others: r-- (read only)
│  │    └─────── group:  r-x (read + execute)
│  └──────────── owner:  rwx (read + write + execute)
└─────────────── type: - = file, d = directory, l = link
```

### Permission values
| Symbol | Number | Meaning |
|---|---|---|
| r | 4 | Read |
| w | 2 | Write |
| x | 1 | Execute |
| - | 0 | No permission |

### Common permission numbers
```
777 → rwxrwxrwx → everyone can do everything (dangerous)
755 → rwxr-xr-x → owner full, others read+execute
644 → rw-r--r-- → owner read+write, others read only
600 → rw------- → owner read+write only
400 → r-------- → owner read only
```

### Change permissions
```bash
# Give owner execute permission
chmod +x script.sh

# Set exact permissions
chmod 755 script.sh
chmod 644 file.txt
chmod 600 privatekey

# Change owner
chown abdu file.txt
chown abdu:abdu file.txt          # change owner and group

# Change group
chgrp developers file.txt
```

---

## VIEWING CONTENT — INTERMEDIATE TO PRO

```bash
# Search inside a file
grep "password" file.txt
grep -i "password" file.txt       # case insensitive
grep -r "password" /etc/          # search recursively
grep -n "password" file.txt       # show line numbers
grep -v "comment" file.txt        # show lines NOT matching

# Count lines, words, characters
wc -l file.txt                    # count lines
wc -w file.txt                    # count words

# Sort file contents
sort file.txt
sort -r file.txt                  # reverse sort
sort -u file.txt                  # unique lines only

# Remove duplicate lines
uniq file.txt

# Cut specific columns
cut -d: -f1 /etc/passwd           # get usernames only
cut -d: -f1,3 /etc/passwd         # username and UID

# Combine sort + uniq
sort file.txt | uniq

# Search with multiple pipes
cat /etc/passwd | grep bash | cut -d: -f1
```

---

## SYSTEM INFO — INTERMEDIATE

```bash
# Who am I?
whoami
id                                # full user and group info

# Who is logged in?
who
w

# System info
uname -a                          # kernel version
hostname                          # machine name
cat /etc/os-release               # OS version

# Disk space
df -h                             # disk usage human readable
du -sh /var/log                   # size of a folder

# Memory
free -h                           # RAM usage

# Running processes
ps aux                            # all processes
ps aux | grep apache              # find specific process
top                               # live process monitor
htop                              # better live monitor

# Network info
ip a                              # IP addresses
ip route                          # routing table
netstat -tuln                     # open ports and listening services
ss -tuln                          # modern netstat replacement
```

---

## TEXT EDITING — INTERMEDIATE

```bash
# nano — easiest editor
nano file.txt
# Ctrl+O → save
# Ctrl+X → exit

# vim — powerful, harder
vim file.txt
# i → insert mode (start typing)
# Esc → go back to command mode
# :w → save
# :q → quit
# :wq → save and quit
# :q! → quit without saving
# dd → delete line
# yy → copy line
# p → paste
```

---

## PIPES AND REDIRECTION — PRO

```bash
# Pipe — send output of one command into another
command1 | command2

# Examples
cat /etc/passwd | grep root
ls -la | grep .txt
ps aux | grep python | grep -v grep

# Redirect output to file
command > file.txt                # overwrite file
command >> file.txt               # append to file

# Redirect errors
command 2>/dev/null               # hide error messages
command 2>&1                      # combine output and errors

# Read from file
command < file.txt

# Practical examples
nmap -sV 192.168.1.0/24 > scan.txt           # save scan results
cat /etc/passwd | cut -d: -f1 > users.txt    # save usernames
grep "Failed" /var/log/auth.log >> fails.txt  # save failed logins
```

---

## ENVIRONMENT VARIABLES — PRO

```bash
# Show all environment variables
env
printenv

# Show specific variable
echo $PATH
echo $HOME
echo $USER

# Set a variable (current session only)
export MYVAR="hello"
echo $MYVAR

# Set API key safely (never hardcode in scripts)
export ANTHROPIC_API_KEY="your-key-here"

# Add to PATH permanently
echo 'export PATH=$PATH:/new/directory' >> ~/.bashrc
source ~/.bashrc
```

---

## USEFUL SHORTCUTS — PRO

```bash
# History
history                           # show command history
!!                                # run last command again
!nmap                             # run last nmap command
Ctrl+R                            # search command history

# Terminal shortcuts
Ctrl+C                            # kill running command
Ctrl+Z                            # pause command (background)
Ctrl+L                            # clear screen
Tab                               # autocomplete
Tab Tab                           # show all options

# Background jobs
command &                         # run in background
jobs                              # list background jobs
fg                                # bring back to foreground
bg                                # continue in background
```

---

## RED TEAM USE — FILE HIERARCHY

```bash
# First thing on a new machine — run these
whoami                            # who am I
id                                # what groups am I in
uname -a                          # what kernel version
cat /etc/passwd                   # list all users
cat /etc/shadow                   # hashed passwords (need root)
cat /etc/crontab                  # scheduled tasks
sudo -l                           # what can I run as sudo
find / -perm -4000 2>/dev/null    # SUID binaries
ls -la /home/                     # other user home dirs
cat ~/.bash_history               # command history
env                               # environment variables
netstat -tuln                     # open ports
ps aux                            # running processes
```
---
*Notes by Abdu | Red Team Journey 2026*