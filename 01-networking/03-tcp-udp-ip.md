# 03 — TCP, UDP & IP Deep Dive
> **Status:** Done | **Date:** 2026-04-01
> **Covered:** Intro to LAN, TCP vs UDP, TCP 3-way handshake, IP basics, Wireshark, subnetting practice

---

## INTRO TO LAN

**LAN = Local Area Network**
A network of devices in one physical location — home, office, school.

### How devices connect in a LAN

| Device | Job |
|---|---|
| **Switch** | Connects devices inside the LAN using MAC addresses |
| **Router** | Connects the LAN to outside networks (internet) |
| **Hub** | Old, sends data to ALL devices (replaced by switch) |
| **Access Point** | Wireless connection into the LAN |

### How a switch works
- Switch learns MAC addresses of connected devices
- When data arrives, switch sends it **only** to the correct device
- Smarter and faster than a hub

### VLAN — Virtual LAN
Split one physical network into multiple logical networks.
- Devices on VLAN 1 cannot talk to VLAN 2 without a router
- Used to separate departments (HR, IT, Finance)
- **Red Team use:** VLAN hopping attack — escape your VLAN into another one

---

## IP — DEEP DIVE

### How IP works
IP is responsible for **addressing and routing** packets across networks.
- Every packet has a **source IP** and **destination IP**
- Routers read the destination IP and forward the packet toward it
- IP itself does not guarantee delivery — that is TCP's job

### IPv4 vs IPv6

| | IPv4 | IPv6 |
|---|---|---|
| Length | 32 bits | 128 bits |
| Format | `192.168.1.1` | `2001:db8::1` |
| Total addresses | ~4.3 billion | 340 undecillion |
| Still used? | Yes, most common | Growing |

### IP Packet — what's inside
```
[ Source IP | Destination IP | TTL | Protocol | Data ]
                                ↑
                    TTL = Time To Live
                    Decreases by 1 at every router
                    Reaches 0 = packet dropped
                    Prevents packets looping forever
```

---

## TCP — Transmission Control Protocol

### What is TCP?
A **reliable, connection-based** protocol at Layer 4 (Transport).
- Guarantees every packet arrives
- Guarantees packets arrive **in order**
- If a packet is lost — TCP resends it
- Used when data must be complete and correct

### TCP 3-Way Handshake
Before any data is sent, TCP opens a connection with 3 steps.

```
Client                        Server
  │                              │
  │ ── SYN ──────────────────→  │   "I want to connect"
  │                              │
  │ ←─ SYN-ACK ──────────────  │   "OK, I'm ready"
  │                              │
  │ ── ACK ──────────────────→  │   "Great, let's go"
  │                              │
  │ ←── DATA flowing both ways ─│
```

**SYN** = Synchronise
**ACK** = Acknowledge
**SYN-ACK** = both at once

### TCP Connection Close — 4 steps
```
Client  →  FIN  →  Server       "I'm done sending"
Client  ←  ACK  ←  Server       "OK noted"
Client  ←  FIN  ←  Server       "I'm done too"
Client  →  ACK  →  Server       "Goodbye"
```

### TCP Flags — important ones

| Flag | Name | Meaning |
|---|---|---|
| SYN | Synchronise | Start a connection |
| ACK | Acknowledge | Confirm received |
| FIN | Finish | Close connection |
| RST | Reset | Force close, something went wrong |
| PSH | Push | Send data immediately |
| URG | Urgent | Priority data |

> **Red Team use:** Nmap SYN scan (`-sS`) sends SYN, waits for SYN-ACK, never completes handshake — stealthier than full connect scan

---

## UDP — User Datagram Protocol

### What is UDP?
A **fast, connectionless** protocol at Layer 4 (Transport).
- No handshake — just sends data immediately
- No guarantee of delivery
- No guarantee of order
- If packets are lost — they are gone, no resend
- Used when speed matters more than reliability

### UDP in action
```
Client  →  Data  →  Server
          (no reply needed, no confirmation)
```

---

## TCP vs UDP — Comparison Chart

| Feature | TCP | UDP |
|---|---|---|
| Connection | Yes — handshake first | No — sends immediately |
| Reliable delivery | ✅ Yes | ❌ No |
| Order guaranteed | ✅ Yes | ❌ No |
| Speed | Slower | Faster |
| Error checking | Yes | Minimal |
| Use cases | HTTP, HTTPS, SSH, FTP, SMTP | DNS, DHCP, VoIP, gaming, video |
| Layer | Transport (Layer 4) | Transport (Layer 4) |

### Simple way to remember
- **TCP** = Phone call — connection first, both sides confirm
- **UDP** = Sending a letter — just send it, don't know if they got it

---

## COMMON PORTS — MEMORISE THESE

| Port | Protocol | Service |
|---|---|---|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP (email) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 8080 | TCP | HTTP alternate |

> **Red Team use:** Port scanning with Nmap reveals open ports → tells you what services are running → tells you what to attack

---

## WIRESHARK — CAPTURING TRAFFIC

### What is Wireshark?
A free tool that captures and shows every packet on your network in real time.
Lets you see exactly what data is being sent and received.

### Install
```bash
sudo apt install wireshark
```

### How to capture
1. Open Wireshark
2. Select your network interface (eth0 or wlan0)
3. Click the blue shark fin button — capture starts
4. Open browser, visit a website
5. Click red square — stop capture

### Key Wireshark filters

| Filter | What it shows |
|---|---|
| `tcp` | All TCP traffic only |
| `udp` | All UDP traffic only |
| `http` | HTTP traffic only |
| `dns` | DNS queries only |
| `ip.addr == 192.168.1.1` | Traffic to/from specific IP |
| `tcp.port == 80` | Traffic on port 80 |
| `tcp.flags.syn == 1` | SYN packets only |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Only the first SYN (no ACK) |

### How to find TCP handshake in Wireshark
1. Capture traffic while opening any website
2. Filter: `tcp.flags.syn == 1`
3. You will see:
   - Packet 1: `SYN` — your computer to server
   - Packet 2: `SYN, ACK` — server replies
   - Packet 3: `ACK` — your computer confirms
4. Right click packet → Follow → TCP Stream — see full conversation

### What each column means
```
No.    — packet number
Time   — timestamp
Source — who sent it
Dest   — who received it
Proto  — protocol (TCP, UDP, DNS...)
Length — size of packet
Info   — summary of what happened
```

> **Red Team use:** Wireshark captures credentials on HTTP (unencrypted), reveals what services are running, shows network topology, finds cleartext passwords

---

## SUBNETTING PRACTICE — WEEK PLAN

Today: **Mixed classes** — /24, /25, /26, /27 combined

### Today's practice targets (subnettingpractice.com)
For every question, answer all 4:
```
1. Network address    → the IP with host part = 0
2. Broadcast address  → the IP with host part = all 255
3. First usable host  → network address + 1
4. Last usable host   → broadcast - 1
```

### This week schedule
| Day | Focus |
|---|---|
| Mon | /24 only |
| **Tue** | **/24, /25, /26, /27 mixed ← TODAY** |
| Wed | All /24 to /28, timed |
| Thu | Random CIDR, speed drill |
| Fri | Full timed test |
| Sat | ipcalc terminal practice |
| Sun | Write 5 subnets from memory |

---

## COMMANDS — QUICK REFERENCE

```bash
# Wireshark from terminal
wireshark &

# tcpdump — command line packet capture (no GUI needed)
sudo tcpdump -i eth0                        # capture all traffic
sudo tcpdump -i eth0 port 80               # only HTTP
sudo tcpdump -i eth0 host 192.168.1.1      # only one IP
sudo tcpdump -i eth0 -w capture.pcap       # save to file
wireshark capture.pcap                      # open saved file

# Check your IP and interface
ip a

# Subnet calculator
ipcalc 192.168.1.0/25
ipcalc 10.10.10.0/27
```

---

## RED TEAM USE — TODAY'S TOPICS

| Topic | Red Team Relevance |
|---|---|
| TCP handshake | Nmap SYN scan (`-sS`) uses incomplete handshake — stealthier |
| UDP | UDP scan (`-sU`) finds hidden services like DNS, SNMP, DHCP |
| Wireshark | Capture cleartext HTTP credentials, see network topology |
| Ports | Open port = potential entry point |
| VLAN | VLAN hopping to reach isolated network segments |

---
*Notes by Abdu | Red Team Journey 2026*