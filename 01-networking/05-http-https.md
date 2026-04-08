# 05 — HTTP & HTTPS
> **Covered:** HTTP methods, status codes, headers, cookies, request/response cycle, HTTPS/TLS, Wireshark HTTP capture, subnetting practice

---

## WHAT IS HTTP?
**HyperText Transfer Protocol** — the rules that browsers and web servers use to communicate.

- Layer 7 (Application) of OSI model
- Port **80** (HTTP) and Port **443** (HTTPS)
- Client sends a **request** → Server sends a **response**
- HTTP is **plain text** — anyone can read it
- HTTPS is **encrypted** — protected with TLS

---

## HTTP REQUEST — STRUCTURE

When your browser visits a website it sends a request like this:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Cookie: session=abc123
Connection: keep-alive
```

### Parts of a request
```
GET /index.html HTTP/1.1     ← Method + Path + Version
Host: example.com            ← Which website
User-Agent: Mozilla/5.0      ← What browser/client
Accept: text/html            ← What format I want back
Cookie: session=abc123       ← My session cookie
                             ← Blank line separates headers from body
[body — only on POST/PUT]    ← Data being sent (forms, JSON)
```

---

## HTTP RESPONSE — STRUCTURE

Server replies like this:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
Set-Cookie: session=abc123; HttpOnly
Server: Apache/2.4

<html>...</html>
```

### Parts of a response
```
HTTP/1.1 200 OK              ← Version + Status code + Message
Content-Type: text/html      ← What format the body is in
Content-Length: 1234         ← Size of the body
Set-Cookie: session=abc123   ← Server sets a cookie on client
Server: Apache/2.4           ← What web server is running

[blank line]

<html>...</html>             ← The actual page content (body)
```

---

## HTTP METHODS

| Method | What it does | Example use |
|---|---|---|
| **GET** | Fetch data from server | Load a webpage, get an image |
| **POST** | Send data to server | Submit a login form, upload file |
| **PUT** | Replace existing data | Update a user profile |
| **PATCH** | Partially update data | Change just one field |
| **DELETE** | Delete data | Delete an account |
| **HEAD** | Like GET but no body | Check if resource exists |
| **OPTIONS** | Ask what methods are allowed | CORS preflight check |

### Red Team use of methods
```
GET  → parameter tampering in URL
POST → intercept with Burp, modify form data
PUT  → upload a web shell if misconfigured
DELETE → delete resources if no auth check
OPTIONS → reveals what server supports
```

---

## HTTP STATUS CODES

### 2xx — Success
| Code | Meaning |
|---|---|
| `200` | OK — request succeeded |
| `201` | Created — new resource made |
| `204` | No Content — success but nothing returned |

### 3xx — Redirection
| Code | Meaning |
|---|---|
| `301` | Moved Permanently — page moved forever |
| `302` | Found — temporary redirect |
| `304` | Not Modified — use cached version |

### 4xx — Client Errors (your fault)
| Code | Meaning |
|---|---|
| `400` | Bad Request — malformed request |
| `401` | Unauthorized — not logged in |
| `403` | Forbidden — logged in but no permission |
| `404` | Not Found — page does not exist |
| `405` | Method Not Allowed — wrong HTTP method |
| `429` | Too Many Requests — rate limited |

### 5xx — Server Errors (server's fault)
| Code | Meaning |
|---|---|
| `500` | Internal Server Error — something broke |
| `502` | Bad Gateway — proxy issue |
| `503` | Service Unavailable — server down |

### Red Team use of status codes
```
401 → try default credentials
403 → try to bypass with header tricks
404 → directory is there but forbidden — worth probing
500 → your input broke the server → potential vulnerability
```

---

## HTTP HEADERS — IMPORTANT ONES

### Request headers
| Header | Meaning |
|---|---|
| `Host` | Which website you want |
| `User-Agent` | Your browser/tool identity |
| `Cookie` | Your session data |
| `Referer` | What page you came from |
| `Authorization` | Login token or credentials |
| `Content-Type` | Format of data you are sending |
| `X-Forwarded-For` | Real IP behind proxy |

### Response headers
| Header | Meaning |
|---|---|
| `Set-Cookie` | Server sets a cookie on your browser |
| `Content-Type` | Format of the response |
| `Server` | Web server software and version |
| `Location` | Where to redirect |
| `X-Powered-By` | Backend technology (PHP, ASP.NET) |

### Red Team use of headers
```
User-Agent    → change to bypass WAF or access mobile version
X-Forwarded-For → spoof your IP address
Referer       → some sites check this for access control
Server        → reveals software version → look for CVEs
X-Powered-By  → reveals tech stack → target specific vulnerabilities
```

---

## COOKIES
Small pieces of data the server stores on your browser.
Used to remember who you are (session management).

### Cookie attributes
| Attribute | Meaning |
|---|---|
| `HttpOnly` | JavaScript cannot read this cookie — protects from XSS |
| `Secure` | Only sent over HTTPS — not sent on HTTP |
| `SameSite` | Controls when cookie is sent cross-site |
| `Expires` | When cookie expires |
| `Path` | Which URL paths the cookie applies to |

### Red Team use of cookies
```
Steal session cookie → log in as that user (session hijacking)
Missing HttpOnly     → steal cookie with XSS
Missing Secure flag  → cookie sent over HTTP → capture with Wireshark
Predictable value    → forge your own cookie
```

---

## HTTP vs HTTPS

| | HTTP | HTTPS |
|---|---|---|
| Port | 80 | 443 |
| Encrypted | ❌ No — plain text | ✅ Yes — TLS encrypted |
| Can Wireshark read it? | ✅ Yes — full content visible | ❌ No — only sees encrypted data |
| Safe for passwords? | ❌ Never | ✅ Yes |
| Certificate needed? | No | Yes — SSL/TLS certificate |

---

## HOW HTTPS WORKS — TLS HANDSHAKE

HTTPS = HTTP + TLS encryption layer.
Before any HTTP data is sent, TLS does a handshake to set up encryption.

```
Client                          Server
  │                               │
  │ ── ClientHello ────────────→ │   "I support these cipher suites"
  │                               │
  │ ←─ ServerHello + Certificate  │   "Use this cipher, here's my cert"
  │                               │
  │ ── Key Exchange ───────────→ │   Share encryption keys securely
  │                               │
  │ ←─ Finished ──────────────── │   "Encryption is ready"
  │                               │
  │ ←── Encrypted HTTP data ────→ │   Normal HTTP now encrypted
```

All HTTP traffic after this point is encrypted — Wireshark shows it as gibberish.

---

## WIRESHARK — HTTP CAPTURE

### Capture HTTP traffic (non-HTTPS site)
```bash
# Start capture
wireshark &

# Or command line
sudo tcpdump -i eth0 port 80 -w http-capture.pcap
```

### Wireshark HTTP filters
```
http                          — all HTTP traffic
http.request                  — only requests
http.response                 — only responses
http.request.method == "GET"  — only GET requests
http.request.method == "POST" — only POST requests
http.response.code == 200     — only 200 OK responses
http.response.code == 404     — only 404 responses
ip.addr == 192.168.1.1        — traffic to/from specific IP
tcp.port == 80                — HTTP port only
```

### How to see full HTTP conversation
1. Capture traffic while visiting an HTTP site
2. Filter: `http`
3. Right click any packet → **Follow → HTTP Stream**
4. You see the full request AND response in plain text
5. Look for: cookies, form data, passwords, tokens

### Comparing HTTP vs HTTPS in Wireshark
```
HTTP site  → Follow HTTP Stream → see everything in plain text
HTTPS site → Follow TCP Stream  → see only encrypted gibberish
```

This is why HTTPS matters — and why HTTP is dangerous on public WiFi.

---

## COMMANDS — QUICK COPY PASTE

```bash
# Capture HTTP with tcpdump
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 port 80 -w http.pcap

# curl — make HTTP requests from terminal
curl http://example.com                          # GET request
curl -I http://example.com                       # HEAD — headers only
curl -X POST http://example.com/login \
     -d "username=admin&password=admin"          # POST request
curl -v http://example.com                       # verbose — see headers
curl -H "User-Agent: MyTool" http://example.com  # custom header
curl -b "session=abc123" http://example.com      # send cookie

# wget — download a page
wget http://example.com

# Check what HTTP methods a server allows
curl -X OPTIONS http://example.com -i
```

---

## RED TEAM USE — TODAY'S TOPICS

| Topic | Red Team Use |
|---|---|
| HTTP methods | PUT upload shell, DELETE bypass, OPTIONS recon |
| Status codes | 403/401 → bypass attempts, 500 → injection worked |
| Headers | Spoof IP, bypass WAF, fingerprint tech stack |
| Cookies | Steal session, XSS cookie theft, forge tokens |
| HTTP in Wireshark | Capture cleartext credentials on HTTP sites |
| HTTPS | Burp Suite acts as proxy to intercept HTTPS |

---
*Notes by Abdu | Red Team Journey 2026*