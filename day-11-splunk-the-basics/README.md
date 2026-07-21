# Day 11: Splunk — The Basics

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ✅ Completed

---

## 📌 Overview

This room takes the SIEM theory from Day 10 and grounds it in **Splunk**, one of the leading SIEM platforms, using a real AttackBox + Lab Machine environment rather than a static simulation.

Key concepts covered:
- **Splunk's three core components:**
  - **Forwarder** — a lightweight agent installed on the monitored endpoint that collects data (web traffic, Windows Event Logs/PowerShell/Sysmon, Linux host logs, DB connection logs) and ships it to Splunk with minimal performance impact.
  - **Indexer** — parses and normalizes incoming data into field-value pairs, categorizes it, and stores it as searchable **events**.
  - **Search Head** — the **Search & Reporting App** where analysts query indexed data using **SPL (Search Processing Language)**, and turn results into tables, pie/bar/column charts, and dashboards.
- **Navigating Splunk:** the Splunk Bar (Messages, Settings, Activity, Help, Find, app-switching), the Apps Panel (default: Search & Reporting), the Explore Splunk panel (Add Data, Splunk Apps, Splunk Docs shortcuts), and the Home Dashboard.
- **Adding data:** Splunk can ingest virtually any log source, transforming raw input into individual searchable events through a guided five-step wizard: **Select Source → Set Source Type → Input Settings → Review → Done**.

The hands-on portion had me start my own AttackBox and Lab Machine, upload a real VPN log file into Splunk, and run SPL queries to answer specific investigative questions about the data.

---

## 🛠️ Tools Used

- **Splunk Enterprise 8.2.6** (Search & Reporting App, Add Data wizard, SPL queries)
- **TryHackMe AttackBox** (browser-based VM environment)

---

## 🪜 Steps Followed

**1. Started the environment and reviewed the Splunk home screen**
Launched the AttackBox and Lab Machine, then opened Splunk's home dashboard — the Explore Splunk panel (Add Data / Splunk Apps / Splunk Docs) and the Search & Reporting app in the left-hand Apps panel.

![Splunk Home Dashboard](screenshots/01-splunk-home-dashboard.png)

**2. Reviewed the reference SPL queries provided for this task**
Before running my own searches, noted the "Useful checks" queries given as a sanity-check reference — confirming total event count, filtering by username, filtering by source IP, and excluding a specific country.

![Reference SPL Checks](screenshots/02-reference-spl-checks.png)

**3. Uploaded the VPN log file**
Downloaded `VPN_logs` (newline-delimited JSON) and used Splunk's **Add Data → Upload** wizard, keeping the auto-detected JSON source type. Reviewed the final configuration before submitting: Uploaded File, `VPNlogs.json`, `_json` source type.

![Add Data - Review](screenshots/03-add-data-review.png)

**4. Ran an initial search to confirm ingestion**
Searched `source="VPNlogs.json" host="1" sourcetype="_json"` and confirmed **2,862 events** ingested, with each event showing raw JSON fields like `Company`, `EventTime`, `Source_Country`, `Source_ip`, `UserName`, `action`, and `port`.

![Initial Search - Raw Event](screenshots/04-initial-search-raw-event.png)

**5. Hit a snag with `| spath` alone**
Tried running just `| spath` on its own and got **0 events / "No results found."** This happened because I'd cleared out the base search (the `source=...` filter) before adding `spath` — without a base search feeding it events, there was nothing for `spath` to parse. This was a useful reminder that `spath` is a **pipe**, not a standalone search — it needs a preceding search to hand it results.

![spath Error - No Results](screenshots/05-spath-error-no-results.png)

**6. Corrected the search and confirmed the total count**
Rebuilt the query properly — `source="VPNlogs.json" host="ip-10-10-40-195" sourcetype="_json" | stats count` — using the actual host value from the environment, and confirmed **2,862 total events**, matching the initial ingestion count.

![Total Count - Corrected](screenshots/06-total-count-corrected.png)

**7. Queried events for user Maleena**
Ran `| spath | search UserName="Maleena" | stats count` and found **60 events**.

![Maleena Count](screenshots/07-maleena-count.png)

**8. Looked up the username tied to a specific source IP**
Ran `| spath | search Source_ip="107.14.182.38" | stats values(UserName) as UserName count` and found the IP belonged to user **Smith**, with **26 events**.

![IP → Username Lookup](screenshots/08-ip-username-lookup.png)

**9. Counted events originating outside France**
Ran `| spath | search Source_Country!="France" | stats count` and found **2,814 events**.

![Non-France Count](screenshots/09-non-france-count.png)

**10. Counted events for a second specific source IP**
Ran `| spath | search Source_ip="107.3.206.58" | stats count` and found **14 events**.

![IP 107.3.206.58 Count](screenshots/10-ip-206-58-count.png)

---

## 🔍 Key Findings

- Total events in the VPN log file: **2,862**
- Events associated with user **Maleena**: **60**
- Username associated with IP `107.14.182.38`: **Smith**
- Events originating from all countries except France: **2,814**
- Events associated with IP `107.3.206.58`: **14**
- `spath` is essential for JSON-formatted logs — without it, Splunk doesn't automatically break the JSON blob into searchable fields like `UserName` or `Source_ip`, even though the raw event data is already ingested and visible.

---

## 💡 Lessons Learned

- The `| spath` troubleshooting moment was actually the most valuable part of this room — reading the SPL pipe left-to-right made it click that each stage only has access to what the previous stage handed it. An empty base search means every downstream pipe has nothing to work with, no matter how correct that pipe's syntax is.
- Splunk's three-component architecture (Forwarder → Indexer → Search Head) maps directly onto the "how EDR works" agent/console model from Day 9, and onto the generic "centralized log collection → normalization → correlation" SIEM features from Day 10 — three different rooms, same underlying pattern, just named differently per product.
- Using the real `source=` and `host=` values that Splunk actually assigned (rather than the placeholder `host="1"` from my first search) mattered — small copy-paste or leftover values from an earlier query can silently make a "working" search return the wrong scope of data.
- Practicing basic SPL commands here (`search`, `stats count`, `stats values(...)  as ... count`, `spath`) gives me a foundation I can build on for more advanced correlation searches in future SOC Level 1 rooms — this felt like the first "real tool" room in the path, versus the simulated dashboards in Days 5–10.

---

*Previous: [Day 10 — Introduction to SIEM](../day10/day10.md)*
*Next: [Day 12 — coming soon]*
