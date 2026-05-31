# Security Projects

Security research completed as part of personal projects and the MSc Security & Network Engineering programme (OS3) at the University of Amsterdam.

Areas Covered: threat hunting, CVE reproduction, protocol security, incident response, IoT/BLE analysis, and infrastructure hardening.

Subject Focus: Security of Systems and Networks, Offensive Technologies, Large Systems, Cybercrime and Forensics, Advanced Security, Web Technology, Software Design, Networking and Routing, Requirements Engineering, Databases, and Computer Programming.

---

## Projects

### Threat Intelligence & Hunting

**blockchain-threat-hunt**
Anomaly detection pipeline over 6.4M Bitcoin transactions to identify adversarial mixer activity — ~22,000 flagged transactions, 97 wallet clusters. `Python` `Isolation Forest` `NetworkX` `MITRE ATT&CK TA0040`

**adsb-injection-detection**
Passive eavesdropping and active aircraft spoofing against ADS-B at 1090 MHz using RTL-SDR and HackRF One. Includes protocol-level detection signatures for IDS tuning. `RTL-SDR` `HackRF` `Python` `MITRE T1565.002`

---

### Incident Response & Vulnerability Research

**CVE-2023-45866-analysis**
Zero-click Bluetooth HID authentication bypass reproduced across 10+ Linux and Android devices. Root cause analysis, attack chain, MITRE mapping, and patch evaluation. `BlueZ` `Raspberry Pi` `MITRE ATT&CK`

---

### IoT & Protocol Security

**nuki-smartlock-ble-analysis**
Security analysis of the Nuki Smart Lock 2.0 — BLE traffic capture, Android app reverse engineering (JADX/Frida), and cryptographic analysis (ECDH, XSalsa20, Poly1305). `BLE` `JADX` `Frida` `Wireshark`

**67-sec-secure-protocol**
Designed a repeater protocol extension (67-SEC) adding HMAC-based message authentication, nonce-derived session keys, and timestamp-based replay resistance. Measured 9% payload overhead and 10% transmission cost — strong authentication at reasonable performance cost. `HMAC` `Cryptography` `Protocol Design` `Python`

---

### Offensive Security

**penetration-testing-and-side-channel**
Penetration testing against DNS, IDS, and web applications; implemented timing and power analysis side-channel attacks to extract cryptographic keys; documented attack methodology and mitigations. `Metasploit` `Burp Suite` `Wireshark` `Nmap` `Side-channel Analysis`

---

### Infrastructure & Cloud Security

**k8s-security-hardening**
Production-grade HA Kubernetes cluster across KVM VMs using Terraform and Ansible; Calico CNI network segmentation, RBAC, Pod Security Standards, zero-trust policy, automated TLS rotation, and MetalLB bare-metal load balancing. Validated resilience via Locust — <1% failure at 6,000 concurrent users. `Kubernetes` `Calico` `Terraform` `Ansible` `KVM`

**cloud-devsecops-pipeline**
Defense-in-depth microservices pipeline with automated SAST/DAST scanning, dependency checks, and IAM hardening on AWS; GitOps migration from monolithic to containerised, reducing deployment time from hours to minutes. `AWS` `Docker` `GitHub Actions` `Terraform`

**docker-ufw-incident-response**
Incident response for a Docker/UFW firewall bypass exposing CouchDB publicly — triage, containment, remediation, automated audit scripts, and runbook. `Docker` `UFW` `Bash` `CouchDB`

---

## About

Ritika Nayak — MSc Security & Network Engineering (OS3), University of Amsterdam | B.Tech Electronics & Telecommunications | Ex-IBM · Ex-Accenture · iSHARE Foundation
`ritikanayakwork@gmail.com` · [LinkedIn](https://www.linkedin.com/in/nayak-ritika/)
