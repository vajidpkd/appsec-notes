# 04 — DNS (Domain Name System)
> **Covered:** What is DNS, how it works step by step, DNS records, nslookup, dig, Wireshark DNS capture, subnetting practice

---

## WHAT IS DNS?
**Domain Name System** — translates human-readable domain names into IP addresses.

Computers talk using IP addresses. Humans remember names.
DNS is the translator between the two.

```
You type:    google.com
DNS returns: 142.250.180.46
Browser connects to that IP
```

> Without DNS you would have to memorise the IP of every website.

---

## HOW DNS RESOLUTION WORKS — STEP BY STEP

```
You type: www.google.com in browser

Step 1 — Check local cache
         Your computer checks if it already knows the IP
         If yes → use it, done
         If no  → continue

Step 2 — Ask Recursive Resolver
         Your ISP or router has a DNS resolver
         It handles the search on your behalf

Step 3 — Ask Root Name Server
         Resolver asks: "who handles .com domains?"
         Root server replies: "ask the .com TLD server"

Step 4 — Ask TLD Name Server
         TLD = Top Level Domain (.com, .org, .net)
         Resolver asks .com server: "who handles google.com?"
         TLD replies: "ask Google's authoritative server"

Step 5 — Ask Authoritative Name Server
         This server knows the exact answer
         Returns the IP: 142.250.180.46

Step 6 — Cache and return
         Resolver saves the answer (cache)
         Returns IP to your computer
         Browser connects

Total time: milliseconds
```

### Simple diagram
```
Browser → Recursive Resolver → Root Server
                             ↓
                        TLD Server (.com)
                             ↓
                     Authoritative Server
                             ↓
                    IP returned → Browser connects
```

---

## DNS RECORDS — ALL TYPES

| Record | Full Name | What it does | Example |
|---|---|---|---|
| **A** | Address | Maps domain → IPv4 address | `google.com → 142.250.180.46` |
| **AAAA** | Quad-A | Maps domain → IPv6 address | `google.com → 2607:f8b0::` |
| **MX** | Mail Exchange | Points to mail server for domain | `gmail.com → mail server` |
| **CNAME** | Canonical Name | Alias — points one domain to another | `www.google.com → google.com` |
| **NS** | Name Server | Tells which server is authoritative for domain | `google.com → ns1.google.com` |
| **TXT** | Text | Stores text info — used for verification, SPF | `"v=spf1 include:..."` |
| **PTR** | Pointer | Reverse lookup — IP → domain name | `142.250.180.46 → google.com` |
| **SOA** | Start of Authority | Admin info about the zone | Serial, refresh, retry times |

### Most important for red team — memorise these
- **A** — find the IP of a target domain
- **MX** — find mail servers (email attacks)
- **TXT** — often leaks internal info, SPF/DKIM config
- **CNAME** — find hidden subdomains
- **NS** — find authoritative name servers (DNS zone transfer target)

---

## nslookup — COMMAND LINE DNS TOOL

Basic tool built into Linux and Windows. Query DNS records from terminal.

### Basic usage
```bash
# Look up A record (default)
nslookup google.com

# Specify record type
nslookup -type=A google.com
nslookup -type=AAAA google.com
nslookup -type=MX google.com
nslookup -type=TXT google.com
nslookup -type=NS google.com
nslookup -type=CNAME www.google.com

# Use a specific DNS server
nslookup google.com 8.8.8.8        # use Google DNS
nslookup google.com 1.1.1.1        # use Cloudflare DNS

# Reverse lookup — IP to domain
nslookup 142.250.180.46
```

### Interactive mode
```bash
nslookup
> set type=MX
> google.com
> exit
```

---

## dig — MORE POWERFUL DNS TOOL

`dig` gives more detail than nslookup. Preferred by red teamers.

### Basic usage
```bash
# A record
dig google.com

# Specific record type
dig google.com A
dig google.com AAAA
dig google.com MX
dig google.com TXT
dig google.com NS
dig google.com CNAME

# Use specific DNS server
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com MX

# Reverse lookup
dig -x 142.250.180.46

# Short answer only (clean output)
dig google.com +short

# All records at once
dig google.com ANY

# Trace full DNS resolution path
dig google.com +trace
```

### Reading dig output
```
; <<>> DiG 9.18 <<>> google.com
;; ANSWER SECTION:
google.com.    300    IN    A    142.250.180.46
               ↑            ↑    ↑
               TTL      record   answer
               (seconds)  type
```

---

## DNS ZONE TRANSFER
A feature that lets DNS servers copy all records from another server.
If misconfigured — anyone can download the entire DNS zone (all records, all subdomains).

```bash
# Attempt zone transfer (red team recon)
dig axfr @ns1.target.com target.com

# With nslookup
nslookup
> server ns1.target.com
> set type=any
> ls -d target.com
```

> **Red Team use:** Zone transfer reveals ALL subdomains, internal hostnames, mail servers, and IPs in one query. Huge recon win if misconfigured.

---

## WIRESHARK — CAPTURE DNS QUERIES

### How to capture DNS traffic
```bash
# Start Wireshark
wireshark &

# Or use tcpdump for command line
sudo tcpdump -i eth0 port 53 -w dns-capture.pcap
```

### Wireshark DNS filters
```
dns                          — show all DNS traffic
dns.qry.name == "google.com" — specific domain query
dns.flags.response == 0      — show only requests
dns.flags.response == 1      — show only responses
udp.port == 53               — DNS over UDP
tcp.port == 53               — DNS over TCP (large responses)
```

### What to look for in a DNS capture
1. Filter: `dns`
2. Find a DNS **Query** packet — your computer asking
3. Find the DNS **Response** packet — server answering
4. Look at:
   - **Query** section — what domain was asked
   - **Answer** section — what IP was returned
   - **TTL** — how long to cache this answer
   - **Protocol** — UDP port 53 (normal) or TCP port 53 (large transfer)

### DNS uses UDP or TCP?
- **UDP port 53** — normal DNS queries (small, fast)
- **TCP port 53** — large responses, zone transfers

---

## DNS ATTACKS — RED TEAM

| Attack | What happens |
|---|---|
| **DNS Zone Transfer** | Download all DNS records if server misconfigured |
| **DNS Enumeration** | Brute force subdomains — find hidden assets |
| **DNS Spoofing** | Fake DNS response — send victim to wrong IP |
| **DNS Cache Poisoning** | Inject fake record into resolver cache |
| **DNS Tunneling** | Hide data/C2 traffic inside DNS queries |
| **Subdomain Takeover** | Claim an abandoned subdomain pointing to dead service |

### Subdomain enumeration (free tools)
```bash
# Install tools
sudo apt install dnsrecon
pip install sublist3r

# dnsrecon — brute force subdomains
dnsrecon -d target.com -t brt

# Sublist3r
python3 sublist3r.py -d target.com

# Online: dnsdumpster.com (free, no install)
```

---

## PUBLIC DNS SERVERS — USEFUL TO KNOW

| IP | Provider | Notes |
|---|---|---|
| `8.8.8.8` | Google DNS | Fast, public |
| `8.8.4.4` | Google DNS | Backup |
| `1.1.1.1` | Cloudflare | Privacy focused, fast |
| `9.9.9.9` | Quad9 | Security filtered |

---

## COMMANDS — QUICK COPY PASTE

```bash
# nslookup
nslookup google.com
nslookup -type=MX google.com
nslookup -type=TXT google.com
nslookup -type=NS google.com

# dig
dig google.com A
dig google.com MX
dig google.com TXT +short
dig google.com +trace
dig @8.8.8.8 google.com

# Zone transfer attempt
dig axfr @ns1.target.com target.com

# Subdomain brute force
dnsrecon -d target.com -t brt

# DNS capture with tcpdump
sudo tcpdump -i eth0 port 53
```

---


*Notes by Abdu | Red Team Journey 2026*
