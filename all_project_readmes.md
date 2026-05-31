<!-- ============================================================ -->
<!-- FILE: blockchain-threat-hunt/README.md                      -->
<!-- ============================================================ -->

# Blockchain Threat Hunt — Tracing Transactions Across Mixers

**TL;DR:** Built an end-to-end anomaly detection pipeline over 6.4M Bitcoin transactions to identify adversarial mixer activity — ~22,000 suspicious transactions flagged, 97 dense wallet clusters identified.

## Context

Bitcoin mixers pool funds from multiple users to obscure transaction trails. They are the primary tool for laundering ransomware payments and evading sanctions. Standard forensics tools lose the trail once funds enter a mixer. This project investigates whether graph analysis and ML can recover it.

Course: Cybercrime and Forensics, University of Amsterdam (OS3)

## What I Did

**Pipeline:**
1. Pulled 1,000 Bitcoin blocks from Google BigQuery (~6.4M UTXO transactions)
2. Built a directed transaction graph using NetworkX
3. Computed a custom suspicion score using four mixer-characteristic features: high in/out-degree, coin fragmentation, rapid multi-hop movement, and dust transfers
4. Applied Isolation Forest (unsupervised ML) — flagged ~22,000 anomalous transactions
5. Fed flagged transactions into Gephi for community detection

## Key Findings

| Metric | Value |
|---|---|
| Transactions analysed | 6,400,000 |
| Flagged by Isolation Forest | ~22,000 |
| Dense wallet clusters (Gephi) | 97 |
| Wallets in suspicious subgraph | 46,000 |
| Modularity score | 0.985 |

- Mixer-like nodes form hub structures with dramatically higher in/out-degree than normal transactions
- 97 communities with strong internal density — consistent with mixer pool behaviour
- Mapped to MITRE ATT&CK TA0040

## Tools & Technologies

`Python` `Google BigQuery` `NetworkX` `scikit-learn (Isolation Forest)` `Gephi` `Matplotlib`

## Defender Takeaway

A SOC analyst could use this pipeline to triage large blockchain datasets without enterprise forensics tooling and generate a ranked suspect list of mixer-involved wallets for further OSINT investigation.

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Cybercrime and Forensics course*


<!-- ============================================================ -->
<!-- FILE: CVE-2023-45866-analysis/README.md                     -->
<!-- ============================================================ -->

# CVE-2023-45866 — Bluetooth HID Authentication Bypass

**TL;DR:** Reproduced a zero-click Bluetooth authentication bypass across 10+ Linux and Android devices. An attacker within Bluetooth range can inject arbitrary keystrokes with no pairing, no user interaction, and no credentials.

## Context

CVE-2023-45866 is a flaw in the Bluetooth HID stack. The Bluetooth specification recommends opening HID control and interrupt channels before authentication for legacy device compatibility. Vendors implemented this faithfully — attackers exploit it to skip authentication entirely by claiming to be a legacy device.

Course: Offensive Technologies, University of Amsterdam (OS3) | February 2026

## Root Cause

Bluetooth spec §5.4.3.3: *"Bluetooth HID devices in Discoverable Mode should allow the HID Control channel and HID Interrupt Channel to be opened without encryption."*

An attacker claims `NoInputNoOutput` capability → skips bonding → lands directly in connected HID state → injects arbitrary keystrokes.

## Attack Flow

```
Attacker (Raspberry Pi 4, BlueZ 5.82)
  │
  ├─ Step 1: Set adapter class to 0x002540 (impersonate keyboard)
  ├─ Step 2: Discover target via SDP on L2CAP channels 17 + 19
  ├─ Step 3: Claim legacy device / NoInputNoOutput → skip bonding
  ├─ Step 4: Connected state reached WITHOUT authentication
  └─ Step 5: Inject arbitrary keystrokes → arbitrary command execution
```

## Devices Tested

| Device | OS Version(s) | Vulnerable? |
|---|---|---|
| Pixel 4a | Android 10, 11, 12, 13 | Yes (pre-patch) |
| Huawei Y5 II | Android 4 | Yes |
| Samsung | Android 10 | Yes |
| Ubuntu | 12, 14, 16 | Yes (discoverable mode) |
| Debian | 12, 13 (patched BlueZ) | No |

## MITRE ATT&CK Mapping

| Technique | ID |
|---|---|
| Bluetooth Exploitation | T1424 |
| Command and Scripting Interpreter | T1059 |
| Initial Access via Physical Proximity | T1200 |

## Key Finding

Many supposedly vulnerable versions were not exploitable in practice — patching varied by vendor and distribution. The tradeoff: the BlueZ patch prevents pairing with certain legacy HID devices, the same compatibility allowance the spec intentionally included.

## Tools & Technologies

`BlueZ` `Raspberry Pi 4` `L2CAP` `Wireshark` `MITRE ATT&CK`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Offensive Technologies course | February 2026*


<!-- ============================================================ -->
<!-- FILE: adsb-injection-detection/README.md                    -->
<!-- ============================================================ -->

# ADS-B Security — Eavesdropping & Aircraft Injection

**TL;DR:** Demonstrated passive eavesdropping and active aircraft spoofing against ADS-B using a ~€30 RTL-SDR dongle and HackRF One — capturing telemetry from ~80 real aircraft and injecting a fake aircraft into live tracking software.

## Context

ADS-B is the global standard for aircraft position reporting. Aircraft broadcast GPS position, altitude, speed, and ID every second at 1090 MHz — with no encryption and no authentication. Anyone with a €30 SDR dongle can receive it. Anyone with a HackRF can spoof it.

Course: Advanced Security Lab, University of Amsterdam (OS3) | March 2026

## Experiments

### Experiment 1 — Passive Eavesdropping

Setup: RTL-SDR dongle → dump1090 → ADS-B Exchange live tracking

| Metric | Value |
|---|---|
| Aircraft detected | ~80 |
| At cruise altitude (35–38k ft) | ~7 |
| RSSI range | -28 dB to -10 dB |

Sample captures: KLM1801 at 5,550 ft / 256 kts, RYR7393 at 38,000 ft / 393 kts.

### Experiment 2 — Message Injection

Setup: HackRF One → crafted ADS-B frames → fake aircraft appears on live tracking map, indistinguishable from real traffic at protocol level.

MITRE ATT&CK: T1565.002 (Transmitted Data Manipulation)

## Detection Signatures (IDS)

```
# Position jump exceeds max aircraft speed
alert: position_delta_km / time_delta_sec > 350

# Instantaneous altitude change
alert: abs(altitude_t - altitude_t-1) > 5000 ft in < 10 seconds

# Duplicate ICAO hex at two positions simultaneously
alert: duplicate_icao_hex AND position_distance > 50 km

# Aircraft ICAO not in registered database
alert: icao_hex NOT IN [registered_aircraft_database]
```

## Tools & Technologies

`RTL-SDR` `HackRF One` `dump1090` `Python` `Matplotlib` `Folium`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Advanced Security Lab course | March 2026*


<!-- ============================================================ -->
<!-- FILE: nuki-smartlock-ble-analysis/README.md                 -->
<!-- ============================================================ -->

# Nuki Smart Lock 2.0 — BLE Security Analysis

**TL;DR:** Security analysis of the Nuki Smart Lock 2.0 using BLE traffic capture, Android app reverse engineering (JADX + Frida), and cryptographic analysis. Found strong ECDH + XSalsa20-Poly1305 implementation with no critical vulnerabilities — remaining risks are local key storage and battery-based DoS.

## Context

Smart locks combine physical and digital security. A compromise doesn't just leak data — it can unlock a door. This project evaluated whether the Nuki Smart Lock 2.0's cryptographic design holds up under static and dynamic analysis.

Course: Security of Systems and Networks, University of Amsterdam (OS3) | April 2026

## Methodology

**1. BLE Traffic Capture**
Enabled Android Bluetooth HCI snoop log → captured lock/unlock operations → extracted `btsnoop_hci.log` → analysed in Wireshark with `btatt` filter, focusing on attribute handle `0x0092` (encrypted write requests).

**2. Static Analysis (JADX)**
Decompiled the Nuki APK to pseudo-Java source. Identified obfuscated encryption classes (e.g. `io.nuki.K30`) as targets for dynamic analysis.

**3. Dynamic Analysis (Frida)**
Hooked obfuscated encryption functions at runtime on a rooted Android device. Used Java reflection to inspect encryption parameters and message contents before and after encryption — confirming the XSalsa20-Poly1305 construction and per-message nonce usage.

## Findings

**Cryptographic design (strong):**
- ECDH key exchange during pairing using Curve25519
- XSalsa20-Poly1305 authenticated encryption for all BLE commands
- Unique nonce per message — prevents replay attacks
- Challenge-response mechanism for command authenticity

**Weaknesses identified:**
- Key material stored in Android application sandbox — exposed if device is rooted
- BLE header metadata visible unencrypted — reveals communication timing patterns
- Battery-based DoS: continuous motor activation can drain battery and disable remote access

**Comparison:** Performs better than August Smart Lock (vulnerable to nonce reuse) and mitigates Bluetooth key negotiation downgrade attacks described by Antonioli et al. (2019).

## Tools & Technologies

`Wireshark` `JADX` `Frida` `ADB` `Android HCI Snoop Log` `Python`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Security of Systems and Networks course | April 2026*


<!-- ============================================================ -->
<!-- FILE: 67-sec-secure-protocol/README.md                      -->
<!-- ============================================================ -->

# 67-SEC — Secure Extension for Repeater Protocol

**TL;DR:** Designed and implemented a security extension for an OS3 repeater protocol adding HMAC-based message authentication, nonce-derived session keys, and timestamp-based replay resistance — achieving strong authentication at 9% payload overhead and 10% transmission cost.

## Context

The existing repeater protocol used a pre-shared secret for authentication but had no replay protection and was vulnerable to packet injection and modification. This project redesigned the security layer from scratch.

Course: Advanced Security Lab, University of Amsterdam (OS3) | February 2026

## Threat Model

The attacker can eavesdrop, inject, replay, and modify packets on an insecure channel. Secrets are not known to the attacker.

Core problems addressed:
- Packet tampering
- Unauthorized packet injection
- Replay attacks / DoS

## Design

**Three-stage iterative design:**

| Version | Properties | Weaknesses |
|---|---|---|
| V1: HMAC only | Authenticity, Integrity | Replay attack, DoS |
| V2: HMAC + Session Keys | Authenticity, Integrity, PFS(?) | Replay attack, DoS |
| V3: HMAC + Session Keys + Timestamps | Authenticity, Integrity, PFS, Replay resistance | DoS, time sensitivity |

**Final packet structure:**
```
[67 68], From, To, Nonce, Timestamp, Tag(K_session, Payload)
```

Where `K_session = Tag(K_shared, nonce)` — session key derived per connection from shared key and nonce.

## Results

| Payload size | Insecure bytes | Secure bytes | Overhead |
|---|---|---|---|
| 5,000 | 5,600 | 6,118 | +9% |
| 10,000 | 11,144 | 12,240 | +9% |
| 20,000 | 22,400 | 24,364 | +9% |
| 30,000 | 33,544 | 36,368 | +9% |

Transmission time overhead: consistent +10% across all payload sizes.

**Conclusion:** Strong authentication and replay resistance at reasonable and predictable performance cost.

## Tools & Technologies

`Python` `HMAC` `Nonce-derived session keys` `Timestamp-based replay resistance`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Advanced Security Lab course | February 2026*


<!-- ============================================================ -->
<!-- FILE: cracking-wep/README.md                                -->
<!-- ============================================================ -->

# Cracking WEP — Why Early Wi-Fi Security Failed

**TL;DR:** Demonstrated full WEP key recovery using the PTW statistical attack — collected 196,668 packets, 73,077 IVs, recovered the WEP-40 key with 100% certainty using aircrack-ng.

## Context

WEP (Wired Equivalent Privacy) was the original Wi-Fi security standard, now cryptographically broken. According to WiGLE scans, 2.7–7% of devices still use it. This project demonstrates exactly why.

Course: Advanced Security, University of Amsterdam (OS3)

## Why WEP Is Broken

- **24-bit IV** = ~16 million possibilities → IV reuse is inevitable on any active network
- **Static 40-bit shared key** — no rotation
- **Insecure RC4 usage** — first 256 keystream bytes not discarded
- **FSM Attack** — probabilistic related-key attack; 5% success rate per key byte with only 60 intercepted packets

## Attack Methodology

Setup:
- OpenWirelessRouTer AP — WEP-40 (Shared Key)
- 2 legitimate clients generating traffic
- Attacker laptop in monitor mode

Attack pipeline:
```
Monitor Mode → Capture Traffic → Collect ~70k IVs → Apply PTW Attack → Recover Key
```

Tools: `airodump-ng` (capture) · `aireplay-ng` (inject) · `aircrack-ng` (crack)

## Results

```
Read 196,668 packets
Got 73,077 out of 70,000 IVs
Starting PTW attack with 73,077 IVs
KEY FOUND! [ 12:34:56:78:90 ]
BSDecrypted correctly: 100%
```

Full key recovered. IV reuse made statistical bias exploitation practical.

## Tools & Technologies

`Kali Linux` `aircrack-ng` `airodump-ng` `aireplay-ng` `Wireshark`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Advanced Security course*


<!-- ============================================================ -->
<!-- FILE: k8s-security-hardening/README.md                      -->
<!-- ============================================================ -->

# Kubernetes Security Hardening — HA Cluster on Bare Metal

**TL;DR:** Designed and deployed a production-grade highly available Kubernetes cluster across 6 VMs on 3 physical servers using Terraform and Ansible. Validated under Locust load testing — zero failures up to 4,600 concurrent users, <1% failure rate at 6,000.

## Context

This project combined infrastructure as code, configuration management, and container orchestration to create a reproducible, fault-tolerant deployment. The goal was to compare orchestration-level scaling (replica counts) versus infrastructure-level scaling (adding worker nodes) under real load.

Course: Large Systems, University of Amsterdam (OS3)

## Architecture

```
kubectl → Load Balancer 1 (HAProxy/Keepalived) ─┐
          Load Balancer 2 (HAProxy/Keepalived) ─┤
                                                 ├─ Control Plane 1
                                                 ├─ Control Plane 2
                                                 ├─ Control Plane 3
                                                 ├─ Worker Node 1
                                                 ├─ Worker Node 2
                                                 └─ Worker Node 3
                                                         │
                                                      MetalLB → External Traffic
```

3 control-plane nodes + 3–7 worker nodes across 3 physical servers. No single point of failure at any layer.

## Implementation

**Infrastructure provisioning (Terraform + cloud-init):**
- 6 VMs provisioned across 3 physical servers, each with 2 vCPUs / 2GB RAM
- cloud-init bootstrapped all nodes into Kubernetes-ready state on first boot

**Cluster automation (Ansible — fully idempotent):**
- Configured HAProxy + Keepalived on load balancers for VIP
- Installed containerd runtime on all nodes
- Ran `kubeadm init` on primary control plane, joined remaining nodes
- Applied Calico CNI for pod networking and NetworkPolicy enforcement
- Deployed MetalLB for bare-metal load balancing with static external IP

**Application:** Google Online Boutique (microservices demo)

## Load Testing Results (Locust)

| Test | Result |
|---|---|
| Zero failures threshold | 4,600 concurrent users |
| <1% failure rate | 6,000 concurrent users |
| <10% failure rate | 10,000 concurrent users |

Key finding: Orchestration-level scaling (more replicas) improved latency under moderate load but hit node resource limits. Infrastructure-level scaling (more worker nodes) provided better scalability at high load by reducing resource contention.

## Tools & Technologies

`Kubernetes v1.30` `Terraform` `Ansible` `Calico CNI` `MetalLB` `HAProxy` `Keepalived` `Locust` `containerd` `kubeadm`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Large Systems course*


<!-- ============================================================ -->
<!-- FILE: cloud-devsecops-pipeline/README.md                    -->
<!-- ============================================================ -->

# Cloud DevSecOps Pipeline — AWS Infrastructure as Code

**TL;DR:** Provisioned AWS infrastructure using Terraform (EC2 instances, security groups, outputs) and explored containerised deployment with Docker and Kubernetes, covering IaC fundamentals, networking, and scaling strategies.

## Context

This was a lab-based exploration of Infrastructure as Code on AWS, covering Terraform provisioning, container runtime management, and Kubernetes cluster setup — building practical IaC skills applied in the larger K8s project.

Course: Large Systems, University of Amsterdam (OS3)

## What Was Covered

**Terraform on AWS:**
- Provisioned EC2 instances and security groups with Terraform
- Used `terraform plan` / `apply` / `output` workflow
- Verified HTTP connectivity between instances

```bash
# Terraform created:
# - 1 Security Group
# - 2 EC2 Instances (54.234.110.14, 54.91.139.128)

curl http://54.234.110.14    # "This page is served by instance number 1."
curl http://54.91.139.128    # "This page is served by instance number 2."
```

**Container runtime (nerdctl/containerd):**
- Ran containers, managed lifecycle, tested HTTP endpoints

**Kubernetes cluster setup:**
- Installed kubeadm v1.30, kubelet, kubectl
- Configured CNI (Calico and Flannel comparison)
- Joined worker nodes via Ansible playbooks
- Resolved UFW/iptables conflicts with cluster networking

## Key Learning: Calico vs Flannel

Calico builds a real Layer-3 routed network using BGP — better performance, scalability, and built-in NetworkPolicy enforcement. Flannel uses VXLAN overlay — simpler, sufficient for small clusters. Calico is the right choice for production.

## Tools & Technologies

`Terraform` `AWS EC2` `Docker` `Kubernetes v1.30` `Calico` `Flannel` `Ansible` `kubeadm`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Lab work, Large Systems course*


<!-- ============================================================ -->
<!-- FILE: docker-ufw-incident-response/README.md               -->
<!-- ============================================================ -->

# Docker/UFW Firewall Bypass — Incident Response

**TL;DR:** Identified and resolved a production security incident where Docker bypassed UFW firewall rules, inadvertently exposing a CouchDB port publicly on a university server. Remediated, added audit scripts, and built a git pre-commit hook to prevent recurrence.

## What Happened

Docker's default behaviour modifies iptables rules directly, bypassing UFW. A CouchDB container was running with a published port that became publicly accessible, unintended by the firewall configuration.

## Response Steps

1. **Identification** — confirmed public exposure via port scan
2. **Containment** — rebound all services to local IP (`127.0.0.1`)
3. **Remediation** — updated Docker daemon config to prevent iptables manipulation
4. **Prevention** — wrote audit script to check for exposed ports; added git pre-commit hook to flag docker-compose files publishing ports without explicit bind address

## Key Lesson

Docker and UFW don't play well together by default. Docker's iptables integration bypasses UFW entirely — `ufw status` can show a port as blocked while it's actually open to the internet.

## Tools & Technologies

`Docker` `UFW` `iptables` `Bash` `CouchDB`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Personal project / incident response*
