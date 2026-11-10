# Active Directory Attack Chain: Simulation & DFIR Investigation

**A self-contained red team / blue team project: compromise a small Active Directory environment using a realistic attack chain, then investigate it like a real incident responder - using three independent evidence sources.**

---

## Introduction

This project simulates a complete, realistic Active Directory compromise - reconnaissance, Kerberoasting, credential theft, and lateral movement to Domain Admin - and then investigates that compromise the way a real incident response team would, using **three independent evidence sources** rather than relying on logs alone:

- **Splunk** (SIEM) : what the logs show
- **Velociraptor** (live endpoint forensics) : what the system's current state shows
- **Volatility 3** (memory forensics) : what was sitting in RAM at the moment of capture

The goal wasn't just to detect an attack, but to **prove** it happened, reconstruct an accurate timeline, and make real containment and recovery decisions.

Everything in this repository was performed inside an isolated, host-only virtual lab against systems I built and own. No real networks, systems, or data were involved.

## Why I Built This

I wanted a project that went beyond SIEM alert triage (which I already had experience with) and into actual digital forensics and incident response - host-level investigation, memory analysis, and evidence correlation across multiple independent sources.

## Attack Chain Summary

| Step | Technique | Outcome |
|---|---|---|
| 1. Reconnaissance | BloodHound AD mapping | Identified `svc-sql` as a Kerberoastable account with a path toward Domain Admin |
| 2. Kerberoasting | Impacket `GetUserSPNs` + Hashcat | Requested and cracked `svc-sql`'s service ticket, recovering its plaintext password |
| 3. Credential Dumping | Mimikatz `sekurlsa::logonpasswords` | Used `svc-sql`'s access to reach a workstation holding a cached Domain Admin session; extracted the NTLM hash from LSASS |
| 4. Lateral Movement | Pass-the-Hash (Impacket `psexec`) | Authenticated directly to the Domain Controller as the compromised Domain Admin account |

**MITRE ATT&CK techniques demonstrated:** T1087.002 (Account Discovery), T1558.003 (Kerberoasting), T1003.001 (OS Credential Dumping: LSASS Memory), T1550.002 (Pass the Hash), T1021.002 (SMB/Windows Admin Shares)

## Investigation Summary

| Evidence Source | What It Confirmed |
|---|---|
| **Splunk** | Kerberoasting ticket requests (Event 4769, RC4 encryption), LSASS process access (Sysmon Event 10), lateral movement logon (Event 4624, Logon Type 3) |
| **Velociraptor** | Prefetch evidence proving Mimikatz execution, process and network state at collection time, confirmed no persistence mechanism was established |
| **Volatility 3** | Live process tree showing `mimikatz.exe` as a child of `powershell.exe`, active network connections from the workstation to the Domain Controller (including `lsass.exe` itself), no evidence of process injection |

**Key finding:** the memory capture caught `lsass.exe` with an active, established network connection to the Domain Controller at the exact moment of acquisition — tying the credential-theft and lateral-movement stages of the attack together in a single piece of evidence, independent of the Splunk logs.

## Full Report

**[Read the complete report here → REPORT.pdf](./Report.pdf)**

The full report contains the lab architecture, detailed methodology for every phase, the complete timeline cross-referencing all three evidence sources, MITRE ATT&CK mapping, containment and recovery actions, hardening recommendations, and an evidence/chain-of-custody log.


## Tools Used

**Environment:** VirtualBox, Windows Server (Domain Controller), Windows 10 (workstation), Kali Linux (attacker), Ubuntu Server (SIEM/forensics host)

**Attack:** BloodHound, Impacket, Hashcat, Mimikatz

**Detection & Forensics:** Splunk Enterprise, Sysmon, Velociraptor, Volatility 3

## Skills Demonstrated

Active Directory & Kerberos internals · Attack path analysis · Credential theft techniques · SIEM detection engineering (SPL) · Live endpoint forensics · Memory forensics · MITRE ATT&CK mapping · Incident timeline reconstruction · Containment & recovery decision-making · Chain of custody / evidence handling · Technical report writing



