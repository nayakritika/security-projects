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
