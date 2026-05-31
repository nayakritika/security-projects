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
Hooked obfuscated encryption functions at runtime on a rooted Android device. Used Java reflection to inspect encryption parameters and message contents before and after encryption — confirming XSalsa20-Poly1305 construction and per-message nonce usage.

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

**Comparison:** Performs better than August Smart Lock (vulnerable to nonce reuse) and mitigates Bluetooth key negotiation downgrade attacks (Antonioli et al., 2019).

## Tools & Technologies

`Wireshark` `JADX` `Frida` `ADB` `Android HCI Snoop Log`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Security of Systems and Networks course | April 2026*
