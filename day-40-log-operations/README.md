# Day 40: Log Operations

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ⚠️ Completed (theory only — no hands-on lab)

---

## 📌 Overview

This room is the operational/management follow-up to [Day 39 — Intro to Logs](https://github.com/philip-ajik/thm-soc-level1-writeups/tree/main/day-39-intro-to-logs), moving from "what is a log" to "how do you actually plan and configure logging in an organisation." No VM this time — it's entirely concept and scenario-based.

The core idea is that log configuration isn't one-size-fits-all — it serves four distinct purposes, and mixing them up leads to either wasted resources or missed requirements:

- **Security** — anomaly/threat detection, authentication logging, integrity and confidentiality assurance.
- **Operational** — proactive status reporting, troubleshooting, capacity planning, and (relevant to one of the quiz questions) **service billing** — i.e. measuring the cost of using a service.
- **Legal** — alignment with standards and regulations (ISO 27001, COBIT, GDPR, PCI DSS, HIPAA, FISMA). The room's PCI DSS example was concrete: active central log management, adequate configuration, 12-month retention (with the last 3 months always searchable), regular security checks, and a yearly audit.
- **Debug** — increasing visibility for developers, mostly confined to test/dev environments rather than production, aimed at enhancing efficiency and speeding up development.

From there, the room walks through the **planning process**: once you know the purpose, you still need to answer a set of base questions before touching a config file — what to log and why, how much detail, how to collect it, how to store and protect it, how to analyse it, and whether you actually have the budget and headcount to sustain it. One planning scenario tied a specific question ("how much do you need to log?") directly to a compliance answer — "as much as mentioned in the PCI DSS requirements" — which is a good illustration of how legal purpose drives operational decisions.

The **configuration dilemma** section frames the tension every team hits: balancing *requirements* (operational and security needs — explicitly non-negotiable) against *aspirations* (more data, more context, threat-hunting capability) within real resource and budget limits. The room splits this into two question sets:

- **Base requirements** (reactive, incident-detection mindset): what happened, when, where, who/what caused it, from which source.
- **Aspirations** (proactive, threat-hunting mindset): more detail, confidence in the finding, what's affected, what happens next, what else needs attention, what to do about it.

The **logging principles** table covers six areas — Collection (log what you need, avoid noise), Format (consistent structure, synced timestamps), Archiving and Accessibility (retention policy, backups), Monitoring and Alerting (actionable alerts, not noise), Security (access controls, encryption, dedicated log management), and Continuous Change (logging sources evolve — train personnel, stay adaptable).

Paired against that is a **challenges** table: data volume and noise, system performance/collection overhead, process-and-archive difficulties (parsing multiple formats, balancing retention against compliance), security of the logs themselves, analysis complexity (correlation across sources, real-time constraints, false positive/negative rates), and a misc bucket covering planning gaps, budget shortfalls, and skills gaps.

The room closes with a genuinely useful real-world cautionary tale: **Windows 7's default logging configuration produced little-to-no useful System/Security/Application event logs when a machine was compromised via EternalBlue (CVE-2017-0144, CVSS 8.1 High)** — a stark example of why "if it works, don't touch it" is the wrong mindset for logging configs. That leads into a **dos-and-don'ts** list: don't log sensitive information, don't hand-roll custom logging without proper planning, don't collect everything and skip analysis, don't chase confirmation bias ("searching for what you want to find" instead of investigating what's actually there) — versus do build a tailored config plan, test at scale, secure the logs, create meaningful (not noisy) alerts, and keep the whole operation continuously maintained and updated.

---

## 🛠️ Tools Used

No hands-on lab this time — this room is entirely scenario-based quiz content, no VM or terminal work involved.

---

## 🪜 Steps Followed

This room didn't have a hands-on component, so there were no steps to walk through — just reading through each task's material and answering the embedded scenario questions as they came up. No screenshots were captured since there was no lab environment to interact with.

---

## 🔍 Key Findings

- Log purpose suited to measuring the cost of using a service: **Operational**
- Log purpose suited to investigating application logs for enhancement and stability: **Debug**
- In a PCI DSS compliance planning scenario, "as much as mentioned in the PCI DSS requirements" answers the question: **How much do you need to log?**
- When negotiating logging budget/scope, the non-negotiable requirements are: **operational and security requirements**
- Deciding which logs to store vs. which portion stays available for analysis maps to the logging principle: **Archiving and Accessibility**
- Continuous errors when transferring a new product's logs to a review platform is an example of the challenge category: **Process and Archive**
- A custom, unplanned logging script that omits logging at some phases is an example of a: **Mistake** (not a best practice)
- Real-world case that justifies continuous log config maintenance: default Windows 7 logging producing no meaningful event logs during an EternalBlue (CVE-2017-0144) compromise — a full-access, CVSS 8.1 High severity incident that went essentially unlogged.

---

## 💡 Lessons Learned

- This room reframed logging as a planning and governance problem before it's a technical one — the four purposes (Security/Operational/Legal/Debug) give a useful lens for *why* a given log source or retention period exists, which is easy to lose sight of when you're just staring at rsyslog configs like in [Day 39](../day39/day39.md).
- The base-requirements-vs-aspirations split maps well onto reactive incident response vs. proactive threat hunting — it's a good mental model for justifying "why do we need more than the minimum" conversations with stakeholders who are budget-focused.
- The EternalBlue/Windows 7 example is a concrete argument for something I've seen asserted abstractly before but not tied to a real CVE: default logging configs are often not fit for security purposes out of the box, and "it hasn't been touched" is not the same as "it's fine."
- Connects directly to the SIEM-02 forwarding configuration from Day 39, where the forwarding line was present but commented out — exactly the kind of planning gap ("lack of implementation," per this room's mistakes list) that this room warns produces blind spots until it's too late.
- Next room in the path is **Log Universe**, which per the material moves into the hands-on side of everything covered conceptually here.
