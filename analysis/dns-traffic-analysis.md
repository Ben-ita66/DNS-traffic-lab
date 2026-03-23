# Part 1 – Capturing & Analyzing DNS Traffic

## 🎯 Objective

Capture live DNS traffic using Wireshark and analyze what happens at each network layer when a DNS query is made.

---

## ⚙️ Setup

### Step 1: Flush DNS Cache

Before capturing, I attempted to flush the DNS cache so Wireshark would capture fresh DNS requests instead of cached results.

```bash
sudo systemd-resolve --flush-caches
```

**Result:** Got an error — Kali Linux wasn't running `systemd-resolved`, so there was no DNS cache service to flush.

> **Insight:** This isn't a problem. Since Kali sends DNS requests directly without caching, every query goes out live over the network — which is exactly what we want to capture.

---

### Step 2: Start Wireshark & Apply Filter

Launched Wireshark and applied the following display filter to isolate DNS traffic:

```
udp.port == 53
```

### Step 3: Trigger a DNS Query

```bash
nslookup www.cisco.com
```

---

## 📦 Analyzing the DNS Query Packet

Selected the packet labeled: **Standard query A www.cisco.com**

### Ethernet II (Layer 2)

| Field | Value |
|---|---|
| Source MAC | `08:xx:xx:xx:xx:xx` |
| Destination MAC | `52:xx:xx:xx:xx:xx` (Default gateway) |

### IPv4 (Layer 3)

| Field | Value |
|---|---|
| Source IP | `10.0.x.x` (My VM) |
| Destination IP | `192.168.x.x` (DNS server / gateway) |

### UDP (Layer 4)

| Field | Value |
|---|---|
| Source Port | Ephemeral (random high port) |
| Destination Port | `53` (DNS) |

### DNS Query Flags

| Flag | Value | Meaning |
|---|---|---|
| Recursion Desired (RD) | `1` | Asking the DNS server to fully resolve the domain on my behalf |

---

## 📦 Analyzing the DNS Response Packet

Source and destination addresses were reversed — exactly as expected in a reply.

### DNS Response Flags

| Flag | Value | Meaning |
|---|---|---|
| Recursion Desired (RD) | `1` | Original request had recursion enabled |
| Recursion Available (RA) | `1` | The DNS server supports and handled recursive resolution |

---

## 🔗 DNS Resolution Chain

The response didn't return a direct IP immediately. Instead, it followed a **CNAME chain**:

```
www.cisco.com
    └── CNAME → alias1.cisco.com
        └── CNAME → alias2.akadns.net      ← Akamai CDN
            └── A Record → [Final IP Address]
```

> **Why CNAMEs?** CDNs like Akamai use CNAME chains to dynamically route users to the nearest/fastest server. This is how large-scale websites handle global traffic efficiently.

---

## 🔄 What Happened Behind the Scenes

1. **ARP** — My VM used ARP to find the MAC address of the gateway (`10.0.x.x`) before sending anything
2. **DNS Query** — Sent a UDP packet to the DNS server on port 53
3. **DNS Response** — Server replied with:
   - A chain of CNAME records
   - The final A record (actual IP address)
4. Additional DNS queries/responses were triggered as part of the full resolution process

---

## 🔐 Security Insight

> DNS traffic over UDP port 53 is **unencrypted by default**.

An attacker on the same network can:
- Passively sniff DNS queries using tools like Wireshark
- See exactly which domains you're resolving
- Potentially manipulate responses (DNS spoofing / cache poisoning)

**Mitigations:** DNS over HTTPS (DoH) or DNS over TLS (DoT) encrypt DNS traffic to prevent this.

---

## 📸 Screenshots

![ARP and DNS Traffic Overview](../screenshots/Screenshot%202026-03-23%20153241.png)

![nslookup Terminal Output](../screenshots/Screenshot%202026-03-23%20154835.png)

![CNAME Chain](../screenshots/Screenshot%202026-03-23%20162920.png)

![ARP and DNS Traffic Overview](../screenshots/Screenshot%202026-03-23%20163921.png)

---

## ✅ Summary

| Concept | Observation |
|---|---|
| DNS uses UDP | Port 53, connectionless |
| Recursion | Client delegates full resolution to the server |
| CNAME chains | Used by CDNs for efficient routing |
| ARP comes first | MAC resolution happens before DNS query is sent |
| DNS is plaintext | Traffic is visible to anyone on the network |
