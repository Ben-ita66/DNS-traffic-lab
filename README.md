# 🔍 DNS Traffic Lab – Exploring DNS with Wireshark

A hands-on lab documenting how DNS traffic works at the packet level using Wireshark inside a Kali Linux VM.

---

## 🧪 Lab Overview

| Detail | Info |
|---|---|
| **Tool** | Wireshark |
| **Environment** | Kali Linux (VM) |
| **Protocol Analyzed** | DNS over UDP (Port 53) |
| **Domain Queried** | `www.cisco.com` |
| **Filter Used** | `udp.port == 53` |

---

## 📁 Structure

```
DNS-traffic-lab/
├── README.md
├── screenshots/        # Wireshark captures from the lab
└── analysis/
    └── dns-traffic-analysis.md   # Full breakdown of findings
```

---

## 🗂️ Parts

- [Part 1 – Capturing DNS Traffic](analysis/dns-traffic-analysis.md)

---

## 🔐 Key Takeaways

- DNS queries and responses travel in plaintext over UDP port 53
- CNAME chains are commonly used by CDNs (like Akamai) to route users efficiently
- An attacker on the same network can passively capture DNS traffic with tools like Wireshark
- Flushing DNS cache before a capture ensures fresh, real queries are sent

---

## 🛠️ Tools Used

- Wireshark
- `nslookup`
- Kali Linux

---

> Part of my ongoing cybersecurity learning journey. Documenting everything in public.  
> Follow along on [LinkedIn](https://www.linkedin.com/in/benitanwabueze) | [X @Ogechee_](https://x.com/Ogechee_)
