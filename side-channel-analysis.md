# Side-Channel Analysis — AES Key Recovery via Correlation Power Analysis

**TL;DR:** Recovered a full 128-bit AES key from a microcontroller using Correlation Power Analysis (CPA) — captured 200 power traces from a ChipWhisperer Nano during live AES encryptions, applied Pearson correlation against Hamming weight predictions, and recovered the complete key `c247f4efdbb2b736da09b7850090b4ba`.

## Context

Side-channel attacks exploit physical leakage (power, timing, EM emissions) rather than mathematical weaknesses in the algorithm. AES is mathematically secure — but hardware implementations leak key-dependent information through power consumption. This project demonstrated that attack practically.

Course: Offensive Technologies, University of Amsterdam (OS3) | January 2026

## Setup

- **Target:** ChipWhisperer Nano running `simpleserial-aes` firmware (AES-128, SimpleSerial v2.1)
- **Firmware:** Compiled using TINYAES128C target against ChipWhisperer hardware
- **Host:** Python script extended to automate trace capture and CPA

## Methodology

### 1. Trace Capture

- Connected ChipWhisperer Nano via USB
- Generated a random 128-bit AES key, sent to device
- For each of 200 traces: generated random plaintext → triggered one AES encryption → captured power consumption (12,000 samples per trace)
- Stored traces and plaintexts as NumPy arrays

Traces showed consistent timing alignment and repeating structure — stable AES execution confirmed, minor amplitude variation visible as expected from data-dependent power consumption.

### 2. Correlation Power Analysis (CPA)

**Attack target:** Output of the AES S-Box in the first encryption round — depends on one known plaintext byte and one unknown key byte, enabling byte-wise analysis.

**Method:** For each of the 16 key bytes:
1. For all 256 possible key candidates, predict power consumption using Hamming weight of the first-round S-Box output
2. Compute Pearson correlation between predicted values and measured power traces at each time sample
3. Select the key candidate with the highest maximum absolute correlation

### 3. Key Recovery

Repeated for all 16 bytes → full AES-128 key recovered:

```
Recovered key: c247f4efdbb2b736da09b7850090b4ba
Verified against encryption key: ✓ exact match
```

## Results

| Observation | Finding |
|---|---|
| Correct key byte | Shows distinct correlation peak at S-Box execution time |
| Incorrect candidates | No consistent peaks — remain near noise floor |
| Traces needed | Correct key ranks poorly at 2 traces; consistently top candidate at 50+ traces |
| Full key recovery | Successful with 200 traces |

The effect of trace count was evaluated at 2, 10, 20, 50, 100, and 200 traces. CPA accuracy increases with trace count — noise averages out and the correct key candidate separates clearly from wrong candidates.

## Defender Takeaway

Software AES is mathematically unbreakable. But hardware running AES leaks the key through power consumption unless countermeasures are applied: masking (randomising intermediate values), hiding (adding noise or randomising execution timing), or using dedicated secure elements with built-in side-channel protection.

## Tools & Technologies

`ChipWhisperer Nano` `Python` `NumPy` `Pearson Correlation` `AES-128` `SimpleSerial v2.1`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Offensive Technologies course | January 2026*
