# 02 — IP Address & Subnet Mask
> **Status:** Done | **Date:** 2026-03-30
> **Covered:** What is IP, IPv4, IPv6 intro, subnet mask, CIDR, Class C, weekly practice plan

---

## WHAT IS AN IP ADDRESS?
A unique address that identifies every device on a network.
Like a postal address — without it, data has nowhere to go.

- Format: 4 numbers separated by dots → `192.168.1.1`
- Each number = 1 byte = 0 to 255
- Two versions: **IPv4** (common today) and **IPv6** (newer, longer)

---

## IPv4 STRUCTURE

```
192  .  168  .  1  .  1
 │       │      │    │
8 bits  8 bits 8 bits 8 bits  =  32 bits total

Each section = octet = 0 to 255
```

---

## IPv6 — BRIEF INTRO
*(Full study later)*

- Created because IPv4 addresses ran out
- Format: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Much longer, much more addresses available
- You will see this on networks — just know it exists for now

---

## PRIVATE IP RANGES — MEMORISE THESE
These are for internal networks only — not reachable from the internet.

| Range | CIDR | Common Use |
|---|---|---|
| `10.0.0.0 – 10.255.255.255` | `/8` | Large enterprise |
| `172.16.0.0 – 172.31.255.255` | `/12` | Medium networks |
| `192.168.0.0 – 192.168.255.255` | `/16` | Home / small office |

> **Red Team use:** If you scan and see these ranges — you are inside a private network. Map it.

---

## SPECIAL IP ADDRESSES

| IP | Name | Meaning |
|---|---|---|
| `127.0.0.1` | Loopback | Yourself — "localhost" |
| `255.255.255.255` | Broadcast | Send to all devices |
| `169.254.x.x` | APIPA | No DHCP found, self-assigned |
| `0.0.0.0` | Any | Listen on all interfaces |

---

## WHAT IS A SUBNET MASK?
Tells a device which part of an IP is the **network** and which part is the **host** (device).

```
IP Address:   192.168.1.100
Subnet Mask:  255.255.255.0

255 = network part  ← fixed, same for everyone on this network
0   = host part     ← changes per device
```

So here:
- Network = `192.168.1` — everyone on this network shares this
- Host = `.100` — this specific device only

---

## HOW TO READ A SUBNET MASK

```
255 . 255 . 255 . 0
 ↓     ↓     ↓    ↓
NET   NET   NET  HOST
```

Devices with the **same network part** = same network = talk directly.
Devices with **different network parts** = different network = need a router.

---

## CIDR NOTATION
A short way to write the subnet mask using a `/number`.
The number = how many bits are the network part.

```
192.168.1.0 / 24
            ↑
            24 bits = network part
            8 bits  = host part
```

| CIDR | Subnet Mask | Usable Hosts |
|---|---|---|
| `/24` | `255.255.255.0` | **254** |
| `/25` | `255.255.255.128` | **126** |
| `/26` | `255.255.255.192` | **62** |
| `/27` | `255.255.255.224` | **30** |
| `/28` | `255.255.255.240` | **14** |
| `/30` | `255.255.255.252` | **2** |

Each time CIDR goes up by 1 → hosts cut in half.

---

## CLASS C — MOST COMMON

Default mask = `255.255.255.0` = `/24`

```
Network address:  192.168.1.0    ← reserved
First host:       192.168.1.1    ← first usable
...
Last host:        192.168.1.254  ← last usable
Broadcast:        192.168.1.255  ← reserved

Total usable hosts = 254
```

### Why 254 not 256?
Always minus 2 — one for network address, one for broadcast. Always.

---

## HOSTS FORMULA

```
Usable hosts = 2^(32 - CIDR) - 2

/24 → 2^8  - 2 = 254
/25 → 2^7  - 2 = 126
/26 → 2^6  - 2 = 62
/27 → 2^5  - 2 = 30
/28 → 2^4  - 2 = 14
/30 → 2^2  - 2 = 2
```

---

## COMMANDS

```bash
# Show your IP and subnet mask
ip a
ip addr show

# Install subnet calculator
sudo apt install ipcalc

# Calculate subnet details instantly
ipcalc 192.168.1.0/24
ipcalc 10.0.0.0/8
ipcalc 192.168.1.0/26
```

---

## RED TEAM USE
When you land on a machine — subnet mask tells you the full range to scan.

```
You find: 10.10.10.5/24
→ Network range: 10.10.10.0 to 10.10.10.255
→ Usable targets: 10.10.10.1 to 10.10.10.254
→ Scan all 254 with Nmap
```

---

## 📅 WEEKLY PRACTICE PLAN — SUBNETTINGPRACTICE.COM

Do this every day this week. Takes only 15 minutes. Do it before bed or after study.

| Day | Practice Task | Site |
|---|---|---|
| Monday | Class C /24 only — network, broadcast, hosts | subnettingpractice.com |
| Tuesday | Class C /24 and /25 — splitting in half | subnettingpractice.com |
| Wednesday | /24, /25, /26, /27 — all four | subnettingpractice.com |
| Thursday | Mixed Class C — random CIDR, speed practice | subnettingpractice.com |
| Friday | Timed mode — answer as fast as you can | subnettingpractice.com |
| Saturday | ipcalc in terminal — verify your answers | Terminal |
| Sunday | Write 5 subnets from memory, no help | Paper or notes |

### For every question on the site, answer these 4 things:
```
1. Network address   → the /xx address itself
2. Broadcast         → last address in range
3. First usable host → network address + 1
4. Last usable host  → broadcast - 1
```

### Target by end of week:
- Answer any /24 to /28 question in under 30 seconds
- No calculator needed for /24 and /25

---

## STUDYING TOMORROW
- [ ] IP addressing deep dive — classes, IPv4 vs IPv6
- [ ] TCP — how it works, 3-way handshake, ports
- [ ] UDP — how it differs from TCP
- [ ] All going into `03-tcp-udp-ip.md`

---
*Notes by Abdu | Red Team Journey 2026*