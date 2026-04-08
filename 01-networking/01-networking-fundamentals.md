# 01 — Networking Fundamentals
> **Covered today:** What is network, what is internet, IP (intro only), MAC address, Vendor/OUI, MAC spoofing, Ping, ICMP, OSI model all 7 layers

---

## WHAT IS A NETWORK?
Two or more devices connected together to share data and resources.

- **LAN** — Local Area Network (home, office)
- **WAN** — Wide Area Network (cities, countries)
- **Internet** — Millions of networks connected globally

---

## WHAT IS THE INTERNET?
A massive global network of networks. All connected using agreed rules called **protocols**.

- Data is broken into small pieces called **packets**
- Packets travel through routers to reach destination
- Packets are reassembled at the other end

---

## IP ADDRESS — INTRO ONLY
*(Full deep dive tomorrow in `02-ip-addressing.md`)*

- IP = **Internet Protocol** address
- Every device needs one to communicate on a network
- Like a postal address for your device
- Two versions exist: **IPv4** and **IPv6**
- Example: `192.168.1.1`

---

## MAC ADDRESS
A **hardware address** burned into a network card (NIC) by the manufacturer.

- Format: `AA:BB:CC:DD:EE:FF` — 6 pairs of hex
- Works at **Layer 2 (Data Link)** of OSI model
- Only used **inside** the local network
- Does NOT travel across the internet — only IP does that

### MAC vs IP — Simple difference
| MAC Address | IP Address |
|---|---|
| Hardware address | Logical address |
| Set by manufacturer | Assigned by network |
| Used inside LAN only | Used across networks |
| Stays the same (normally) | Can change |

---

## VENDOR / OUI
The **first 3 bytes** of every MAC address = the manufacturer (vendor).
This is called the **OUI — Organisationally Unique Identifier**.

```
MAC:   00:1A:2B  :  3C:4D:5E
       └──────┘     └──────┘
        VENDOR       DEVICE ID
        (OUI)        (unique per device)
```

### Examples
| OUI | Vendor |
|---|---|
| `00:50:56` | VMware |
| `B8:27:EB` | Raspberry Pi |
| `00:1A:2B` | Cisco |

> **Red Team use:** Look up any MAC at `macvendors.com` to identify what devices are on a network during recon

---

## MAC ADDRESS SPOOFING — Linux
Changing your MAC to a fake one — to hide identity or bypass MAC filtering.

### Check current MAC
```bash
ip link show
```

### Method 1 — Manual
```bash
sudo ip link set eth0 down
sudo ip link set eth0 address AA:BB:CC:DD:EE:FF
sudo ip link set eth0 up
ip link show eth0        # verify
```

### Method 2 — macchanger
```bash
sudo apt install macchanger

sudo macchanger -r eth0                        # random MAC
sudo macchanger -m AA:BB:CC:DD:EE:FF eth0     # specific MAC
sudo macchanger -s eth0                        # show current vs original
sudo macchanger -p eth0                        # restore original
```

> **Note:** MAC spoofing only works on local network. Beyond your router, only IP matters.
>
> **Red Team use:** Bypass WiFi MAC filtering, avoid logging, impersonate trusted device

---

## PING & ICMP PROTOCOL

### What is ICMP?
**Internet Control Message Protocol** — tests if a device is reachable, sends error messages.
- Lives at **Layer 3 (Network)**
- Not for sending real data — only for diagnostics
- Ping uses: **Echo Request** (you send) → **Echo Reply** (target responds)

```
You  →  ICMP Echo Request  →  Target
You  ←  ICMP Echo Reply    ←  Target

Got reply  = host is alive
No reply   = host down OR firewall blocking ICMP
```

### Ping flags

| Flag | Meaning | Example |
|---|---|---|
| `-c` | Count — how many packets then stop | `ping -c 4 google.com` |
| `-i` | Interval — seconds between packets | `ping -i 0.5 google.com` |
| `-s` | Size — packet size in bytes | `ping -s 1000 google.com` |
| `-4` | Force IPv4 only | `ping -4 google.com` |
| `-6` | Force IPv6 only | `ping -6 google.com` |

### Commands
```bash
ping -c 4 192.168.1.1                    # 4 packets only
ping -c 10 -i 0.2 192.168.1.1           # 10 packets, fast
ping -s 1472 192.168.1.1                 # large packet size
ping -4 -c 3 google.com                  # force IPv4
ping -6 -c 3 google.com                  # force IPv6
ping -c 5 -i 0.5 -s 500 192.168.1.1    # combine flags
```

> **Red Team use:** Quick host discovery. If no reply — ICMP may be blocked by firewall, try other methods.

---

## OSI MODEL — ALL 7 LAYERS

**Open Systems Interconnection** — breaks network communication into 7 layers. Each layer has one job.

- Sending: data goes **DOWN** — Layer 7 → Layer 1
- Receiving: data goes **UP** — Layer 1 → Layer 7

### Quick Reference Table

| # | Layer | Job | Protocols | Data Unit |
|---|---|---|---|---|
| 7 | Application | User apps, interface | HTTP, FTP, SSH, DNS, SMTP | Data |
| 6 | Presentation | Translate, encrypt, compress | SSL/TLS, JPEG, ASCII | Data |
| 5 | Session | Open and manage sessions | NetBIOS, RPC | Data |
| 4 | Transport | End-to-end delivery, ports | TCP, UDP | Segment |
| 3 | Network | Routing using IP | IP, ICMP, ARP | Packet |
| 2 | Data Link | Local delivery using MAC | Ethernet, WiFi | Frame |
| 1 | Physical | Raw bits, cables, radio | Cables, WiFi signal | Bits |

---

### Each Layer Explained Simply

**Layer 7 — Application**
What the user sees. Apps live here.
Protocols: HTTP, HTTPS, SSH, FTP, DNS, SMTP
Example: You open a browser and type a URL
Red Team: SQLi, XSS, file upload, RCE — most attacks live here

---

**Layer 6 — Presentation**
Translates data so both sides understand each other. Handles encryption and compression.
Protocols: SSL/TLS, JPEG, ASCII, Unicode
Example: HTTPS encrypts your data at this layer
Red Team: SSL stripping attacks

---

**Layer 5 — Session**
Opens, keeps alive, and closes sessions between two devices.
Protocols: NetBIOS, RPC
Example: Staying logged into a website while browsing multiple pages
Red Team: Session hijacking, session fixation

---

**Layer 4 — Transport**
Breaks data into segments and delivers it end-to-end. **Ports live here.**
Protocols: TCP (reliable, ordered), UDP (fast, no guarantee)
Example: TCP makes sure all your file arrives correctly
Red Team: Port scanning, SYN flood

**TCP vs UDP:**
| TCP | UDP |
|---|---|
| Reliable | Fast |
| Connection first (handshake) | No connection needed |
| HTTP, SSH, FTP | DNS, video, gaming |

---

**Layer 3 — Network**
Routes packets between different networks using IP addresses.
Protocols: IP, ICMP, ARP
Devices: Router
Example: Your ping travels from India to a server abroad through many routers
Red Team: IP spoofing, ICMP flood, traceroute recon

---

**Layer 2 — Data Link**
Delivers frames between devices on the same local network using MAC addresses.
Protocols: Ethernet, WiFi (802.11)
Devices: Switch
Example: Your laptop talks to your home router
Red Team: ARP spoofing, MAC spoofing, VLAN hopping

---

**Layer 1 — Physical**
The actual physical transmission — electrical signals, light pulses, radio waves.
Examples: Ethernet cable, fiber optic, WiFi radio signal
Devices: Hub, cable, network card (hardware)
Red Team: Cable tapping, rogue WiFi access point, hardware keylogger


## WHERE ATTACKS HAPPEN — OSI CHEAT SHEET

| Layer | Common Attacks |
|---|---|
| 7 Application | SQLi, XSS, CSRF, RCE, file upload |
| 6 Presentation | SSL stripping |
| 5 Session | Session hijacking |
| 4 Transport | Port scan, SYN flood |
| 3 Network | IP spoofing, ICMP flood |
| 2 Data Link | ARP spoof, MAC spoof, VLAN hop |
| 1 Physical | Cable tap, rogue AP, hardware implant |

---

## COMMANDS — QUICK COPY PASTE

```bash
# Show MAC
ip link show

# Spoof MAC
sudo ip link set eth0 down
sudo ip link set eth0 address AA:BB:CC:DD:EE:FF
sudo ip link set eth0 up

# macchanger random
sudo macchanger -r eth0

# Ping 4 packets
ping -c 4 <target>

# Ping fast with size
ping -c 10 -i 0.5 -s 500 <target>

# Force IPv4 or IPv6
ping -4 -c 3 <target>
ping -6 -c 3 <target>
```
---
*Notes by Abdu | Red Team Journey 2026*
