# Blockchain Threat Hunt — Tracing Transactions Across Mixers

**TL;DR:** Built an end-to-end anomaly detection pipeline over 6.4M Bitcoin transactions to identify adversarial mixer activity — ~22,000 suspicious transactions flagged, 97 dense wallet clusters identified.

## Context

Bitcoin mixers pool funds from multiple users to obscure transaction trails. They are the primary tool for laundering ransomware payments and evading sanctions. Standard forensics tools lose the trail once funds enter a mixer. This project investigates whether graph analysis and ML can recover it.

Course: Cybercrime and Forensics, University of Amsterdam (OS3)

## What I Did

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

## Defender Takeaway

A SOC analyst could use this pipeline to triage large blockchain datasets without enterprise forensics tooling and generate a ranked suspect list of mixer-involved wallets for further OSINT investigation.

## Tools & Technologies

`Python` `Google BigQuery` `NetworkX` `scikit-learn (Isolation Forest)` `Gephi` `Matplotlib`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Cybercrime and Forensics course*
