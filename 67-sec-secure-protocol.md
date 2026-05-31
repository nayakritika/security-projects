# 67-SEC — Secure Extension for Repeater Protocol

**TL;DR:** Designed and implemented a security extension for an OS3 repeater protocol adding HMAC-based message authentication, nonce-derived session keys, and timestamp-based replay resistance — achieving strong authentication at 9% payload overhead and 10% transmission cost.

## Context

The existing repeater protocol used a pre-shared secret for authentication but had no replay protection and was vulnerable to packet injection and modification. This project redesigned the security layer from scratch.

Course: Advanced Security Lab, University of Amsterdam (OS3) | February 2026

## Threat Model

The attacker can eavesdrop, inject, replay, and modify packets on an insecure channel. Secrets are not known to the attacker.

Core problems addressed: packet tampering, unauthorized packet injection, replay attacks.

## Design Evolution

| Version | Properties | Weaknesses Remaining |
|---|---|---|
| V1: HMAC only | Authenticity, Integrity | Replay attack, DoS |
| V2: HMAC + Session Keys | Authenticity, Integrity, PFS | Replay attack, DoS |
| V3: HMAC + Session Keys + Timestamps | Authenticity, Integrity, PFS, Replay resistance | DoS, time sensitivity |

**Final packet structure:**
```
[67 68], From, To, Nonce, Timestamp, Tag(K_session, Payload)
```

Session key derived per connection: `K_session = Tag(K_shared, nonce)`

## Results

| Payload size (bytes) | Insecure | Secure | Overhead |
|---|---|---|---|
| 5,000 | 5,600 | 6,118 | +9% |
| 10,000 | 11,144 | 12,240 | +9% |
| 20,000 | 22,400 | 24,364 | +9% |
| 30,000 | 33,544 | 36,368 | +9% |

Transmission time overhead: consistent +10% across all payload sizes.

Strong authentication and replay resistance at a predictable, reasonable performance cost.

## Tools & Technologies

`Python` `HMAC` `Nonce-derived session keys` `Timestamp-based replay resistance`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Advanced Security Lab course | February 2026*
