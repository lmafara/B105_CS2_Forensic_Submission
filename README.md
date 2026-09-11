# Digital Forensic Investigation — Nitroba University Harassment Email Case

**Case ID:** CIP-B105-CS2_C11-26-DFIT-17305_NASIRU-LAWAL
**Examiner:** NASIRU LAWAL
**Evidence Source:** Nitroba University Harassment Scenario (Dr. Simson Garfinkel, Naval Postgraduate School / [digitalcorpora.org](https://digitalcorpora.org))

## Overview

This repository contains the forensic examination of a network packet capture (`nitroba.pcap`) submitted as evidence in a harassment complaint. Chemistry 109 instructor **Lily Tuckrige** reported receiving a threatening electronic message warning her to stop teaching, sent via a web-based anonymous messaging service (`willselfdestruct.com`).

The investigation reconstructs the website interaction flow, recovers the verbatim threatening message, identifies the sending device by IP and MAC address, and correlates that device against identity-bearing artefacts (authenticated web sessions) found elsewhere in the same capture — cross-referenced against the Chemistry 109 class roster.

## Repository Structure

```
├── Report/                  Final PDF/DOCX forensic report (9-section structure)
├── Hashes/                  Evidence file hash manifest (MD5/SHA-256)
├── Timeline/                Integrated UTC timeline
├── Evidence_Register/       Evidence tracking table (ID, source, reference, purpose)
├── Screenshots/             Numbered figures referenced from the report
├── Command_Outputs/         Saved tshark query output (.txt)
└── README.md
```

## Tools Used

| Purpose | Tool |
|---|---|
| Packet capture analysis | `tshark` (Wireshark CLI) |
| Evidence integrity | `md5sum`, `sha256sum` |
| Pattern extraction | `grep`, `sed` |
| Platform | Kali Linux (VirtualBox VM), shared folder for evidence transfer |

## Key Findings (Summary)

- **Evidence:** `nitroba.pcap` — 94,410 packets, capture window 2008-07-22 01:51:07–06:13:47 UTC
- **Website:** `www.willselfdestruct.com` confirmed at IP **69.25.94.22**
- **Sending client:** IP **192.168.15.4**, MAC **00:17:f2:e2:c0:ce**
- **Threatening message submitted:** 2008-07-22 **06:04:24 UTC** (frame 83600), addressed to `lilytuckrige@yahoo.com`, reading in part *"you can't hide from us. Stop teaching. Start running."*
- **Identity artefacts on the same device:**
  - Facebook session → `beth@bethr.org` (137 occurrences)
  - Gmail/Calendar session → `jcoachj@gmail.com` (113 occurrences), active ~3.5 minutes *before* the threatening submission
- **Roster comparison:** `jcoachj@gmail.com` is consistent with **"Johnny Coach"** on the Chemistry 109 roster; `beth@bethr.org` matches no roster name and remains an unresolved secondary identity cue
- **Confidence:** High confidence on device/message identification; moderate confidence on session-identity linkage; attribution to a single named individual is limited by the network being shared and unsecured

Full findings, evidence tables, and the calibrated final conclusion are documented in [`Report/`](./Report).

## Reproducing This Analysis

```bash
# Evidence integrity
md5sum nitroba.pcap
sha256sum nitroba.pcap

# Locate the harassment traffic
tshark -r nitroba.pcap -Y 'http contains "willselfdestruct"' \
  -T fields -e frame.number -e frame.time -e ip.src -e ip.dst -e http.request.uri

# Extract the POST content
tshark -r nitroba.pcap -Y 'frame.number==83600' \
  -T fields -e urlencoded-form.key -e urlencoded-form.value

# Identify the client MAC
tshark -r nitroba.pcap -Y 'frame.number==83600' -T fields -e eth.src

# Search that MAC's traffic for identity-bearing emails
tshark -r nitroba.pcap -Y 'eth.addr==00:17:f2:e2:c0:ce and http' \
  -T fields -e http.cookie | grep -oE '[A-Za-z0-9._%+-]+(%40|@)[A-Za-z0-9.-]+\.[A-Za-z]{2,}'
```

See `Command_Outputs/` for the complete set of queries and captured output used throughout this examination.

## Scope Note

Four additional files (`mycapture.pcap`, `mycapture10_10_81.75.pcap`, `mycapture10_10_81_0.pcap`, `ubalt.pcap`) were supplied alongside the case evidence but were reviewed and **excluded as out of scope** — they belong to a separate training exercise on an unrelated subnet and are not part of this case's evidentiary chain.

## Disclaimer

This is an academic digital forensics exercise using a publicly available training dataset (the Nitroba scenario) created for DFIR education and tool-testing purposes. No real individuals or live systems are involved. Identity attribution in this report reflects packet-level evidence only and is explicitly qualified with confidence levels and alternative explanations — it does not constitute proof of legal culpability.

## Author

**NASIRU LAWAL**
Case ID: CIP-B105-CS2_C11-26-DFIT-17305_NASIRU-LAWAL
