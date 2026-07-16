# Ransomware Attack Simulation in a Controlled Lab

**Course:** Computer Networking 2 — Spring 2026

**Author:** Ali Kashi Pazha (Student ID: 40121723)

**Instructor:** Dr. Yaeghoobi

A hands-on lab that simulates a ransomware infection inside an isolated virtual machine using the [ShinoLocker](https://shinolocker.com/) educational ransomware simulator, then forensically analyzes the attack using **Wireshark** (network layer) and **Process Monitor** (endpoint/file-system layer).

> ⚠️ **Educational purpose only.** ShinoLocker is a safe, reversible ransomware *simulator* built for security training and research. It behaves like real ransomware (C2 communication, file encryption, ransom note) but always provides the decryption key. This project was conducted entirely inside an isolated VM — never run this kind of tooling on a production or personal machine.

---

## 📋 What This Project Covers

1. **Lab setup** — building an isolated VM, installing Wireshark and Procmon, and configuring a ShinoLocker payload.
2. **Cryptography fundamentals** — plaintext/ciphertext/key relationships and how ransomware weaponizes standard encryption.
3. **Ransomware lifecycle** — infection, C2 communication, encryption, and extortion phases, mapped onto ShinoLocker's own behavior.
4. **Network traffic analysis (Wireshark)** — DNS queries, C2 domain/IP discovery, TLS handshake behavior, and identification of primary/fallback C2 infrastructure.
5. **Endpoint analysis (Procmon)** — file system activity, dropped payloads, command-line arguments linking the encryption key to the C2 panel, and anti-forensic self-deletion behavior.
6. **Indicators of Compromise (IoCs)** — a consolidated table of domains, IPs, file artifacts, and process names observed during the simulation.
7. **Mitigation & defense strategies** — DNS sinkholing, firewall/IPS blocking rules, endpoint hardening (SMB/RDP), offline backups, and native Windows Defender protections (Controlled Folder Access, behavioral monitoring, etc.).

The full write-up, methodology, and analysis are available in the [project report](./docs/AliKashiPazha_40121723_CN2_PROJECT.pdf).

---

## 📁 Repository Structure

```
AliKashiPazha_40121723_CN2_PROJECT/
├── docs/
│   └── AliKashiPazha_40121723_CN2_PROJECT.pdf   # Full report (setup, analysis, findings)
│
├── lab/
│   ├── ProcessMonitor.zip                       # Sysinternals Process Monitor
│   ├── ShinoLocker.rar                          # ShinoLocker simulator build
│   └── Wireshark.3.6.8.x86.rar                  # Wireshark installer (x32)
│
├── logs/
│   ├── procmon/
│   │   ├── procmon_shinolocker_log.PML          # Full Procmon capture
│   │   └── procmon_SetRenameInformationFile_log.PML  # Filtered rename/encryption events
│   └── wireshark/
│       └── wireshark_log.pcapng                 # Full network capture
│
├── screenshots/                                 # Step-by-step lab screenshots
│
└── target_files/                                # Sample "victim" files used during the simulation
    ├── dummy.docx
    ├── dummy.jpg
    ├── dummy.mp4
    ├── dummy.png
    ├── dummy.pptx
    └── dummy.txt
```

---

## 🧪 Lab Methodology (Summary)

1. Build an isolated VM and disable AV/firewall protections *inside the sandbox only*.
2. Install Wireshark and Procmon; build a ShinoLocker payload configured to target common file extensions.
3. Populate a `target_files` folder with sample documents, images, and media.
4. Start Wireshark and Procmon capture, then run the ShinoLocker payload.
5. Observe the encryption phase and confirm target files become inaccessible (`.shino` extension).
6. Retrieve the decryption key from the ShinoLocker C2 panel using the HOST ID / TRANSACTION ID, then decrypt and restore the files.
7. Stop captures and save the Wireshark (`.pcapng`) and Procmon (`.PML`) logs for analysis.

Full step-by-step instructions with screenshots are in Section 1 of the [report](./docs/AliKashiPazha_40121723_CN2_PROJECT.pdf).

---

## 🔍 Key Findings

| Category | Artifact | Notes |
|---|---|---|
| C2 Domain | `shinolocker.com` | Primary C2, TLSv1.2 |
| C2 Domain | `shinosec.com` | Fallback C2, TLSv1.3 |
| C2 IP | `188.166.237.163` | Resolved via DNS A record |
| C2 IP | `128.199.83.111` | Resolved via DNS A record |
| Staging File | `%TEMP%\<random>.lst` | Target file list written before encryption |
| Malicious Process | `<random>.exe` (e.g. `cKPTrxih.exe`) | Dropped payload performing actual encryption/rename |
| File Extension | `.shino` | Appended to encrypted files |

The report details how command-line arguments passed to the dropped payload (`[Action_Flag] [Encryption_Key] [Victim_ID] [Target_File_Path]`) directly correlate with the key and victim ID shown on the ShinoLocker web panel — providing concrete proof of the local-to-C2 key exchange.

---

## 🛠️ Tools Used

- **[ShinoLocker](https://shinolocker.com/)** — ransomware simulator (safe, reversible)
- **[Wireshark](https://www.wireshark.org/)** — network protocol analyzer
- **[Process Monitor (Procmon)](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)** — Sysinternals endpoint monitoring tool

---

## 🛡️ Mitigation Highlights

- DNS filtering / sinkholing to block known C2 domains before key exchange occurs
- Firewall/IPS rules built from extracted IoCs (domains + IPs)
- Disabling unnecessary SMB/RDP services to reduce lateral movement risk
- Maintaining genuine air-gapped/offline backups (VM snapshots are not a substitute)
- Relying on native OS defenses (Controlled Folder Access, behavioral/heuristic AV, cloud-delivered protection) that were intentionally disabled for this lab

See Section 4 of the report for the full defense strategy breakdown.

---

## 📄 License / Disclaimer

This repository is for **educational and academic purposes only**, submitted as coursework for Computer Networking 2. All activity was performed in an isolated, sandboxed virtual machine using a purpose-built ransomware *simulator* (not live malware). Do not use any material here to target real systems.
