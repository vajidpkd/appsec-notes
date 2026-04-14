# 02 — Linux Commands — Zero to Confident
> **Chapter:** 2 of 2 | **Date:** 2026-04-09
> **Read Chapter 1 first — file hierarchy**

---

## WHERE COMMANDS LIVE

```
/bin          → basic commands    ls, cat, cp, mv, rm
/sbin         → admin commands    reboot, iptables
/usr/bin      → installed tools   nmap, python3, git
/usr/local/bin → manual installs  your custom tools
```

```bash
which nmap          # find where command lives
which python3
whereis nmap        # full path and docs location
echo $PATH          # folders Linux searches for commands
```

---

## CHAPTER 1 — NAVIGATION

```bash
pwd                 # where am I right now
ls                  # list files
ls -l               # list with details
ls -a               # show hidden files
ls -la              # details + hidden
ls -lh              # human readable sizes
ls -lt              # newest files first

cd /etc             # go to /etc
cd ~                # go to home directory
cd ..               # go up one level
cd -                # go back to previous directory

clear               # clear screen
```

---

## CHAPTER 2 — FILE OPERATIONS

```bash
# CREATE
touch file.txt               # create empty file
touch a.txt b.txt c.txt      # create multiple
mkdir folder                 # create folder
mkdir -p a/b/c               # create nested folders

# COPY
cp file.txt /tmp/            # copy to folder
cp file.txt newname.txt      # copy and rename
cp -r folder/ /tmp/          # copy folder

# MOVE AND RENAME
mv file.txt /tmp/            # move file
mv old.txt new.txt           # rename file
mv folder/ /tmp/             # move folder

# DELETE
rm file.txt                  # delete file
rm -f file.txt               # force delete
rm -r folder/                # delete folder
rm -rf folder/               # force delete folder

# VIEW
cat file.txt                 # print whole file
cat -n file.txt              # print with line numbers
less file.txt                # page by page (q to quit)
head file.txt                # first 10 lines
head -n 20 file.txt          # first 20 lines
tail file.txt                # last 10 lines
tail -n 20 file.txt          # last 20 lines
tail -f /var/log/syslog      # follow live updates
```

---

## CHAPTER 3 — SEARCHING

```bash
# SEARCH INSIDE FILES
grep "word" file.txt         # find word in file
grep -i "word" file.txt      # case insensitive
grep -r "word" /etc/         # search entire folder
grep -n "word" file.txt      # show line numbers
grep -v "word" file.txt      # lines NOT matching
grep -l "word" /etc/*        # show matching filenames only

# FIND FILES
find / -name "passwd"                    # find by name
find /etc -name "*.conf"                 # find .conf files
find /home -type f                       # files only
find /home -type d                       # folders only
find / -size +10M                        # bigger than 10MB
find / -mtime -7                         # modified last 7 days
find / -perm -4000 2>/dev/null           # SUID files ← red team
find / -writable -type d 2>/dev/null     # writable folders ← red team
find / -user root -type f 2>/dev/null    # root owned files

# FASTER SEARCH
locate passwd               # search locate database
updatedb                    # update database
```

---

## CHAPTER 4 — TEXT PROCESSING

```bash
# SORT
sort file.txt               # sort A to Z
sort -r file.txt            # reverse Z to A
sort -n file.txt            # sort by number
sort -u file.txt            # sort and remove duplicates

# UNIQUE
uniq file.txt               # remove consecutive duplicates
sort file.txt | uniq        # remove all duplicates
uniq -c file.txt            # count occurrences

# CUT COLUMNS
cut -d: -f1 /etc/passwd     # first column (usernames)
cut -d: -f1,3 /etc/passwd   # columns 1 and 3
cut -d, -f2 data.csv        # CSV second column

# COUNT
wc file.txt                 # lines words characters
wc -l file.txt              # lines only
wc -w file.txt              # words only

# REPLACE
sed 's/old/new/' file.txt           # replace first match
sed 's/old/new/g' file.txt          # replace all matches
sed -i 's/old/new/g' file.txt       # replace and save

# AWK — COLUMN PROCESSING
awk '{print $1}' file.txt           # first column
awk -F: '{print $1}' /etc/passwd    # usernames
awk '{print $1, $3}' file.txt       # columns 1 and 3
```

---

## CHAPTER 5 — PIPES AND REDIRECTION

```bash
# PIPE — output goes into next command
cat /etc/passwd | grep root
ls -la | grep .txt
ps aux | grep python

# REDIRECT TO FILE
command > file.txt          # write to file (overwrite)
command >> file.txt         # append to file
command 2>/dev/null         # hide error messages
command > file.txt 2>&1     # save output and errors

# PRACTICAL EXAMPLES
nmap -sV 192.168.1.1 > scan.txt
cat /etc/passwd | cut -d: -f1 > users.txt
grep "Failed" /var/log/auth.log | wc -l
cat /etc/passwd | grep bash | cut -d: -f1
```

---

## CHAPTER 6 — PERMISSIONS

```bash
# VIEW
ls -la
# -rwxr-xr--
# │└─┬─┘└─┬─┘└─┬─┘
# │  owner group others
# r=4  w=2  x=1

# COMMON VALUES
# 777 = rwxrwxrwx = everyone everything (dangerous)
# 755 = rwxr-xr-x = owner full others read+execute
# 644 = rw-r--r-- = owner read+write others read
# 600 = rw------- = owner only
# 400 = r-------- = read only

# CHANGE PERMISSIONS
chmod 755 file.txt
chmod 644 file.txt
chmod +x script.sh           # add execute
chmod -x script.sh           # remove execute
chmod -R 755 folder/         # recursive

# CHANGE OWNER
chown abdu file.txt
chown abdu:abdu file.txt     # owner and group
chown -R abdu folder/

# CHANGE GROUP
chgrp developers file.txt
```

---

## CHAPTER 7 — USERS AND GROUPS

```bash
# WHO AM I
whoami                       # username
id                           # user ID and groups
id abdu                      # info about user

# USER INFO
cat /etc/passwd              # all users
cat /etc/shadow              # password hashes (root only)
cat /etc/group               # all groups

# SWITCH USER
su abdu                      # switch to abdu
su -                         # switch to root
sudo command                 # run one command as root
sudo -l                      # what can I run as sudo
sudo su                      # become root

# MANAGE USERS
sudo useradd -m abdu         # add user with home folder
sudo passwd abdu             # set password
sudo userdel -r abdu         # delete user and home

# MANAGE GROUPS
sudo groupadd team           # create group
sudo usermod -aG team abdu   # add user to group
groups abdu                  # show user groups
```

---

## CHAPTER 8 — PROCESSES

```bash
# VIEW PROCESSES
ps aux                       # all processes
ps aux | grep apache         # find specific
top                          # live monitor
htop                         # better live monitor

# FIND PROCESS ID
pgrep apache
pidof apache2

# KILL PROCESS
kill 1234                    # kill by PID
kill -9 1234                 # force kill
killall apache2              # kill by name

# BACKGROUND JOBS
command &                    # run in background
jobs                         # list jobs
fg                           # bring to foreground
Ctrl+Z                       # pause job
Ctrl+C                       # kill job
```

---

## CHAPTER 9 — NETWORKING

```bash
# IP AND INTERFACES
ip a                         # show all IPs
ip link show                 # show MAC addresses
ip route                     # routing table

# CONNECTIVITY
ping -c 4 google.com
traceroute google.com

# PORTS AND CONNECTIONS
netstat -tuln                # listening ports
ss -tuln                     # modern version
ss -anp                      # all with process names

# DNS
nslookup google.com
dig google.com
dig google.com MX
cat /etc/hosts               # local DNS
cat /etc/resolv.conf         # DNS server

# DOWNLOAD
wget http://example.com
curl http://example.com
curl -I http://example.com   # headers only
curl -O http://example.com/file

# SSH
ssh user@192.168.1.1
ssh -p 2222 user@192.168.1.1
scp file.txt user@192.168.1.1:/tmp/
```

---

## CHAPTER 10 — SYSTEM INFO

```bash
# SYSTEM
uname -a                     # kernel version
hostname                     # machine name
cat /etc/os-release          # OS info
uptime                       # how long running
date                         # current date and time

# HARDWARE
lscpu                        # CPU info
free -h                      # RAM usage
df -h                        # disk space
du -sh /var/log              # folder size
lsblk                        # disk layout

# LOGS
last                         # login history
who                          # who is logged in
history                      # command history
cat ~/.bash_history          # full history
```

---

## CHAPTER 11 — SHORTCUTS AND TRICKS

```bash
# HISTORY
history                      # all commands
!!                           # run last command again
!nmap                        # run last nmap command
Ctrl+R                       # search history

# TERMINAL SHORTCUTS
Ctrl+C                       # kill command
Ctrl+Z                       # pause command
Ctrl+L                       # clear screen
Ctrl+A                       # go to start of line
Ctrl+E                       # go to end of line
Tab                          # autocomplete
Tab Tab                      # show all options

# MULTIPLE COMMANDS
command1 ; command2          # run both
command1 && command2         # run second only if first works
command1 || command2         # run second only if first fails
```

---

## RED TEAM CHECKLIST — RUN ON EVERY NEW MACHINE

```bash
whoami                                   # who am I
id                                       # my groups
uname -a                                 # kernel version
cat /etc/os-release                      # OS info
hostname                                 # machine name
ip a                                     # network info
cat /etc/passwd                          # all users
sudo -l                                  # sudo permissions
cat /etc/crontab                         # scheduled tasks
find / -perm -4000 2>/dev/null           # SUID binaries
find / -writable -type d 2>/dev/null     # writable dirs
ls -la /home/                            # other users
cat ~/.bash_history                      # command history
env                                      # environment vars
netstat -tuln                            # open ports
ps aux                                   # running processes
ls -la /tmp                              # files in /tmp
```

---

## QUICK REFERENCE CARD

```
NAVIGATE     pwd  cd  ls
CREATE       touch  mkdir
READ         cat  less  head  tail
COPY/MOVE    cp  mv  rm
SEARCH       grep  find  locate
TEXT         sort  uniq  cut  wc  sed  awk
PIPES        |  >  >>  2>/dev/null
PERMS        chmod  chown  chgrp
USERS        whoami  id  su  sudo
PROCESS      ps  top  kill  jobs
NETWORK      ip  ping  netstat  ssh  curl
SYSTEM       uname  df  free  history
```

---
*Notes by Abdu | Red Team Journey 2026 | Chapter 2 of 2*