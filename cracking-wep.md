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

Pipeline:
```
Monitor Mode → Capture Traffic → Collect ~70k IVs → Apply PTW Attack → Recover Key
```

Tools: `airodump-ng` (capture) · `aireplay-ng` (inject) · `aircrack-ng` (crack)

## Result

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
