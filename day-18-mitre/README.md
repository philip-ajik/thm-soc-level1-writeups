# Day 18: MITRE

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ⚠️ Completed (theory only — no hands-on lab captured, research-based room)

---

## 📌 Overview

MITRE is a not-for-profit that runs R&D across cyber security, AI,
healthcare, and space systems, and this room was a tour of its cyber
security frameworks and tools — MITRE ATT&CK, the CAR Knowledge Base,
D3FEND, and a handful of newer/adjacent projects.

**ATT&CK** started in 2013 as MITRE's answer to a real problem:
documenting and standardizing the tactics, techniques, and procedures
(TTPs) used by APT groups, broken into three layers — **Tactic** (the
adversary's goal, the "why"), **Technique** (how they achieve it), and
**Procedure** (the specific implementation). It began Windows-focused
and has since expanded into Enterprise (covering macOS, Linux, cloud),
plus dedicated Mobile and ICS matrices. The **ATT&CK Matrix** lays
tactics across the top with techniques (and sub-techniques) nested
underneath, and the **ATT&CK Navigator** lets you annotate and explore
it visually — e.g. Reconnaissance (tactic) → Active Scanning (technique)
→ Scanning IP Blocks / Vulnerability Scanning / Wordlist Scanning
(sub-techniques). Each technique page carries an ID, description,
real-world procedure examples (groups, software, campaigns),
mitigations, detections, and references.

What makes ATT&CK actually useful day-to-day is that it gives the whole
security community one shared vocabulary — the same technique often
gets called different things across different reports, and ATT&CK's
standard naming plus unique IDs fixes that. It also bridges threat
intel and defense directly: a report can describe *what* an attacker
did, but mapping that to ATT&CK TTPs is what turns it into actual
detection logic, queries, and playbooks. Different roles lean on it
differently — CTI teams map observed behavior into actionable industry
profiles, SOC analysts use it for alert context and prioritization,
detection engineers map their SIEM/EDR rules against it for coverage,
incident responders use it to visualize an attack timeline, and red/
purple teams build emulation plans around known group techniques. The
room's practical example was **Mustang Panda (G0129)** — a group that
favors phishing for initial access, scheduled tasks for persistence,
file obfuscation for defense evasion, and ingress tool transfer for
C2 — analyzed via its ATT&CK group page and Navigator layer. The
follow-on exercise applied the same approach to a hypothetical: as an
analyst at an aviation-sector org migrating to the cloud, using the
ATT&CK Groups page to identify which APT groups target aviation and
what gaps that leaves in defensive coverage.

**CAR (Cyber Analytics Repository)** is where ATT&CK gets turned into
actual detections — a set of validated, ATT&CK-mapped analytics with
pseudocode plus tool-specific implementations (Splunk, EQL, LogPoint),
and sometimes unit tests to validate an analytic works. The example
covered was **CAR-2020-09-001 (Scheduled Task – File Access)**, which
lays out how a scheduled task's file access can be detected across a
few different SIEM query languages.

Where ATT&CK explains how attacks happen, **D3FEND** (Detection,
Denial, and Disruption Framework Empowering Network Defense) is its
defensive mirror — a matrix of 7 defensive tactics with their own
techniques and IDs, explaining specific countermeasures (e.g.
**Credential Rotation, D3-CRO** — regularly rotating passwords so
stolen credentials can't be reused) and tying each one back to the
ATT&CK techniques it defends against.

The room closed with a few adjacent MITRE projects: the **Adversary
Emulation Library** (step-by-step plans to mimic specific real threat
groups, maintained by the Center for Threat-Informed Defense),
**Caldera** (an automated adversary emulation tool for testing
detection and incident response in a controlled environment, usable by
both red and blue teams), and two newer domain-specific matrices —
**AADAPT** (adversary tactics targeting blockchain, smart contracts,
and digital wallets) and **ATLAS** (adversary tactics targeting AI/ML
systems).

---

## 🛠️ Tools Used

- No hands-on lab or AttackBox this time — this room was framework research and reading, using the live ATT&CK website, ATT&CK Navigator, CAR, and D3FEND matrices referenced in the material rather than a deployed VM

---

## 🪜 Steps Followed

No screenshot-based walkthrough for this one — the room was theory and
research-based rather than a hands-on lab, so there's no step sequence
to document beyond working through the ATT&CK Matrix, Navigator,
Mustang Panda's group page, CAR's example analytic, and the D3FEND
matrix as described in the Overview above.

---

## 🔍 Key Findings

- TTP breakdown: **Tactic** = the goal/why, **Technique** = the how, **Procedure** = the specific implementation
- Mustang Panda (G0129) TTP profile: phishing for initial access, scheduled tasks for persistence, file obfuscation for defense evasion, ingress tool transfer for C2
- CAR-2020-09-001 (Scheduled Task – File Access) was the example analytic reviewed, with pseudocode plus Splunk/LogPoint implementations
- D3FEND example: Credential Rotation (D3-CRO) as the defensive counter to credential-reuse attacks
- Adjacent MITRE projects worth remembering: Adversary Emulation Library, Caldera, AADAPT (blockchain/digital assets), ATLAS (AI/ML systems)

---

## 💡 Lessons Learned

- ATT&CK's real value isn't the matrix itself, it's the shared vocabulary — I've noticed the same technique get called different things across different write-ups in this series already, and having a consistent ID to anchor to would have made cross-referencing earlier days easier.
- D3FEND being explicitly built as ATT&CK's mirror (attack technique ↔ defensive technique) is a cleaner way to think about "what do I actually do about this TTP" than I'd been doing informally in earlier days' Lessons Learned sections.
- This ties everything from Days 13, 16, and 17 together retroactively: Day 13's Account Manipulation categorisation, Day 16's Kill Chain phases, and Day 17's Unified Kill Chain phases were all informally doing what ATT&CK/CAR/D3FEND do formally — naming a behavior, then mapping a detection and a defense to it. Worth going back through those TTPs and attaching actual ATT&CK IDs where I can.
- The aviation-sector CTI exercise was a good reminder that threat modeling isn't generic — "what APT groups target us" is a completely different research question depending on the sector, and ATT&CK's Groups page is built specifically to answer that per-industry.
