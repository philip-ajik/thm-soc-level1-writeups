# Day 37: NetworkMiner

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ✅ Completed

---

## 📌 Overview

This room covers NetworkMiner, an open-source Network Forensic Analysis Tool (NFAT) that runs on Windows, Linux, macOS, and FreeBSD. It can operate in two ways: as a passive sniffer that fingerprints hosts, sessions, and open ports without sending any traffic, or — its main use case — as a PCAP parser that reassembles files, images, credentials, and messages from captured traffic for offline analysis.

The room's own comparison sums up the tool's role well: **quickly overview the pcap with NetworkMiner to grab the "low hanging fruit," then go deeper with Wireshark or tcpdump.** It's explicitly not meant to be a primary sniffer — the sniffing feature is Windows-only and isn't as reliable as a dedicated tool like Wireshark.

**Pros:** OS fingerprinting (via the Satori and p0f/mac-ages GitHub projects), easy file extraction, credential grabbing, cleartext keyword parsing, and a fast overall traffic overview.
**Cons:** not built for active sniffing, struggles with large pcaps, has limited filtering, and isn't suited to manual packet-level investigation.

The room walks through the main tabs — **Hosts** (IP/MAC/OS/ports/sessions), **Sessions**, **DNS**, **Credentials** (Kerberos, NTLM, RDP cookies, HTTP cookies/requests, IMAP, FTP, SMTP, MS SQL — crackable with Hashcat or John the Ripper), **Files**, **Images**, **Parameters**, **Keywords**, and **Messages**.

It also highlights a version-jump worth remembering: NetworkMiner **1.6.x** exposes raw frame-by-frame detail (including sequence numbers) and a cleartext tab that 2.7 dropped, while **2.7** added MAC-address-conflict correlation and a much richer Parameters tab. Because of this, the lab VM ships both versions side by side, and a couple of the exercise questions specifically required switching back to 1.6 to get at frame-level detail.

---

## 🛠️ Tools Used

- NetworkMiner 2.7.2 (primary version used throughout)
- NetworkMiner 1.6.1 (used specifically for frame-level sequence number detail)
- Provided lab pcaps: `mx-3.pcap`, `mx-4.pcap`, `mx-7.pcap`, `mx-9.pcap`, `case1.pcap`, `case2.pcap`

---

## 🪜 Steps Followed

**1. Loaded `mx-3.pcap` and checked the Files tab**
Opened the pcap and looked at the Files tab first — only two files were extracted from this capture, `ads.720DB241.html` and `download.html`.

![Files tab overview](screenshots/01-mx3-files-tab-overview.png)

**2. Pulled the file metadata to get the total frame count**
Right-clicked the loaded file in the Case Panel and selected "Show Metadata." The frame count field confirmed **460 total frames**.

![Metadata dialog showing frame count](screenshots/02-mx3-metadata-frame-count.png)

**3. Checked which hosts share a MAC address with 145.253.2.203**
Sorted the Hosts tab by IP and expanded 145.253.2.203, whose MAC is `FEFF20000100`. Two other entries in the list — 216.239.59.104 and 216.239.59.99 — carry that same MAC, giving **2 IP addresses** sharing it.

![Hosts list, first MAC match](screenshots/03-mx3-hosts-list-mac-match.png)
![Hosts list scrolled, second MAC match](screenshots/04-mx3-hosts-scrolled-second-mac-match.png)

**4. Checked packet counts and the web server banner for host 65.208.228.223**
Selected the `65.208.228.223 [www.ethereal.com]` host and read off Sent/Received — **72 packets sent**. Expanded Host Details on the same entry and found the web server banner under TCP 80: **Apache**.

![Host 65.208.228.223 packet counts](screenshots/05-mx3-host-65-208-228-223-packet-counts.png)
![Same host, maximized window](screenshots/06-mx3-host-65-208-228-223-maximized.png)
![Web server banner: Apache](screenshots/07-mx3-webserver-banner-apache.png)

**5. Loaded `mx-4.pcap` and pulled NTLM credentials for host 02694W-WIN10**
Switched to the Credentials tab. The extracted username was `#B\Administrator`, and the full NTLMv2 challenge-response hash was captured alongside it — enough to hand off to a cracking tool like Hashcat or John the Ripper.

![Credentials tab, NTLM hash for 02694W-WIN10](screenshots/08-mx4-credentials-ntlm-hash.png)
![Credentials tab context menu](screenshots/09-mx4-credentials-context-menu.png)

**6. Loaded `mx-7.pcap`**
Started a fresh NetworkMiner session and loaded the larger `mx-7.pcap` capture (this one took a while to parse — over 100x more files than `mx-3`).

![Loading mx-7.pcap](screenshots/10-mx7-loading-pcap.png)

**7. Identified the Linux distro referenced in frame 63602's file**
Frame 63602 itself didn't return a hit, so — per the room's own hint — I fell back to frame 63075 instead. Filtered the Files tab for `63075`, found `index.EE08FE3A.txt`, opened its file details, and read the hex/ASCII dump. The mirror path referenced `centos`, confirming the distro as **CentOS**.

![Files tab filtered to frame 63075](screenshots/11-mx7-files-filtered-frame-63075.png)
![File details context menu](screenshots/12-mx7-file-details-context-menu.png)
![File details hex dump](screenshots/13-mx7-file-details-hexdump.png)
![CentOS confirmed in hex dump](screenshots/14-mx7-file-details-centos-highlighted.png)

**8. Found the name and surname referenced in frame 76469's file**
Same pattern as the step above — frame 76469 had no direct hit, so I used the fallback frame 75942 instead. Its file details showed the text **"Ned Flanders"**.

![index.html file details, Ned Flanders highlighted](screenshots/15-mx7-index-html-ned-flanders.png)

**9. Found the source address of `ads.bmp.2E5F0FD9[1].bmp`**
Browsed the Images tab (which is full of near-identical 1x1 tracking-pixel ad images) then filtered the Files tab directly for the specific filename. Source host came back as **80.239.178.187**.

![Images tab, ads.bmp thumbnail grid](screenshots/16-mx7-images-tab-ads-bmp-grid.png)
![Files tab filtered to the bmp, source IP shown](screenshots/17-mx7-files-filtered-ads-bmp-source-ip.png)

**10. Found the frame number of the possible TLS anomaly**
Checked the Anomalies tab, which listed two "TLS data boundary is not on a TLS record boundary" errors, at frames 36255 and 73073. The room's expected answer was **frame 36255**.

![Anomalies tab, TLS boundary errors](screenshots/18-mx7-anomalies-tls-boundary-error.png)

**11. Loaded `mx-9.pcap` and identified which platform sent the "You have more..." email**
Opened the Messages tab and located the email with the subject starting "You have more friends on Facebook than you think." Opening it showed a "facebook.com" link in the body, confirming the sending platform as **Facebook**.

![Messages tab, full list](screenshots/19-mx9-messages-tab-list.png)
![Facebook email selected](screenshots/20-mx9-facebook-email-selected.png)

**12. Found Branson Matheson's email address**
Selected the message from Branson Matheson in the same Messages list — the From field read `Branson Matheson <branson@sandsite.org>`, giving **branson@sandsite.org**.

![Branson Matheson email selected](screenshots/21-mx9-branson-matheson-email.png)

**13. Loaded `case1.pcap` and checked the OS of host 131.151.37.122**
Moving into the exercises, the Hosts tab showed four hosts total. Expanding 131.151.37.122 showed p0f/NetSA fingerprinting it as Windows NT 4.0 SP1+ (100%), and the Satori TCP fingerprint agreed: **Windows - Windows NT 4**.

![Hosts tab, four hosts](screenshots/22-case1-hosts-tab-overview.png)
![Host 37.122 OS fingerprint, Satori highlighted](screenshots/23-case1-host-37-122-os-satori.png)

**14. Investigated the session between 131.151.37.122 and 131.151.32.91**
Drilled into the session list under Host Details. The session over TCP 1065 showed the client (*.32.91) sending **192 bytes** to the server.

![Session detail, port 1065, 192 bytes from client](screenshots/24-case1-session-port-1065-192-bytes.png)

**15. Investigated the session between 131.151.37.122 and 131.151.32.21**
Same host details view, different session — over TCP 143 (IMAP), the server (*.37.122) sent back **20769 bytes** to the client.

![Session detail, port 143, 20769 bytes from server](screenshots/25-case1-session-port-143-20769-bytes.png)

**16. Found the sequence number of frame 9**
NetworkMiner 2.7 doesn't expose per-frame sequence numbers, so — per the hint — I switched to **NetworkMiner 1.6.1**, which still has a Frames tab. Expanding frame 9's TCP layer showed **Sequence Number = 2AD77400**.

![NetworkMiner 1.6.1, frame 9 sequence number](screenshots/26-case1-networkminer16-frame9-sequence.png)

**17. Counted the detected content types**
Back on 2.7, filtered the Parameters tab for "content type." The values cycling through were `text/plain` and `multipart/mixed` — **2 distinct content types** detected.

![Parameters tab, content-type values](screenshots/27-case1-parameters-content-type-count.png)

**18. Loaded `case2.pcap` and found the USB device's brand name**
This one took real digging. Filtered the Files tab for "usb" and found `hi-Speed-usb2.0_ax88772.htm`, served from `asix.com.tw`. Opening the file details and reading the embedded product title confirmed the brand as **ASIX**.

![Files tab filtered to "usb"](screenshots/28-case2-files-filtered-usb.png)
![Source/destination host columns, asix.com.tw](screenshots/29-case2-source-destination-host-asix.png)
![File details, ASIX brand confirmed in page title](screenshots/30-case2-file-details-asix-brand-confirmed.png)

**19. Found the source IP of the fish image**
Filtered the Files tab for "fish" and found `Crazy-Fishing.jpg`, sourced from **50.22.95.9**.

![Files tab filtered to "fish"](screenshots/31-case2-files-filtered-fish-source-ip.png)

**20. Found the password for homer.pwned.se@gmx.com**
First attempt: filtered the Parameters tab directly for the email address, which only surfaced repeated POP3 "+OK mailbox ... has 1 message" responses — no password. Switched approach and went to the Credentials tab instead, sorting by username to find the account: the POP3 credentials showed password **spring2015**.

![Parameters tab, failed search attempt](screenshots/32-case2-parameters-failed-password-search.png)
![Credentials tab, password found](screenshots/33-case2-credentials-tab-password-found.png)

**21. Found the DNS query of frame 62001**
Filtering the Files tab directly for "62001" came back empty. Switched to the Sessions tab, filtered the same way, and located the client session for that frame (192.168.0.51 → server, port 23972). Cross-referencing that with the DNS tab showed the query: **pop.gmx.com**.

![Files tab filtered to 62001, no results](screenshots/34-case2-files-filtered-62001-empty.png)
![Sessions tab, frame 62001 located](screenshots/35-case2-sessions-frame-62001.png)
![DNS tab, query confirmed as pop.gmx.com](screenshots/36-case2-dns-frame-62001-query.png)

---

## 🔍 Key Findings

- Total frames in `mx-3.pcap`: **460**
- IPs sharing a MAC with 145.253.2.203: **2** (216.239.59.99, 216.239.59.104)
- Packets sent by host 65.208.228.223: **72**
- Web server banner on 65.208.228.223: **Apache**
- Extracted username, `mx-4.pcap` (02694W-WIN10): **#B\Administrator**
- Extracted NTLMv2 challenge-response hash for that user (crackable via Hashcat/John)
- Linux distro referenced in `mx-7.pcap` (frame 63075 fallback): **CentOS**
- Name/surname referenced in `mx-7.pcap` (frame 75942 fallback): **Ned Flanders**
- Source address of `ads.bmp.2E5F0FD9[1].bmp`: **80.239.178.187**
- Frame with the TLS boundary anomaly: **36255**
- Platform behind the "You have more..." email in `mx-9.pcap`: **Facebook**
- Branson Matheson's email address: **branson@sandsite.org**
- Full OS of host 131.151.37.122 (`case1.pcap`): **Windows - Windows NT 4**
- Bytes sent by client *.32.91 over TCP 1065: **192**
- Bytes sent back by server *.37.122 over TCP 143: **20769**
- Sequence number of frame 9 (found via NetworkMiner 1.6.1): **2AD77400**
- Detected content types in `case1.pcap`: **2** (`text/plain`, `multipart/mixed`)
- USB device brand in `case2.pcap`: **ASIX**
- Phone model referenced in `case2.pcap`: **Lumia 535**
- Source IP of the fish image: **50.22.95.9**
- Password for homer.pwned.se@gmx.com: **spring2015**
- DNS query in frame 62001: **pop.gmx.com**
- **Pattern worth calling out:** almost every "hard" answer in this room wasn't found on the first, most obvious tab — the room deliberately nudges you toward cross-referencing tabs (Files ↔ Sessions ↔ DNS, Parameters ↔ Credentials) when a direct filter comes up empty, which mirrors real triage work more than a scripted walkthrough would.

---

## 💡 Lessons Learned

- NetworkMiner's real strength is speed of triage, not depth — it's a "what's here at a glance" tool, and the room's own advice (overview with NetworkMiner, then dig with Wireshark) is a good habit to actually follow rather than just note down.
- Version differences genuinely matter here, not just as trivia: I had to switch to **1.6.1** for the frame 9 sequence number and again as a hint-driven fallback elsewhere, since 2.7 traded away raw frame/cleartext detail for better MAC correlation and richer Parameters data.
- When a direct filter search comes up empty (Files tab for frame numbers, Parameters tab for the GMX password), the fix wasn't to give up on NetworkMiner — it was to pivot to a different tab that holds the same underlying session/credential data from a different angle. That cross-tab habit (Files ↔ Sessions ↔ DNS, Parameters ↔ Credentials) was the single biggest recurring lesson of this room.
- The room's built-in fallback hints (e.g. "if frame 63602 shows nothing, check 63075") are a deliberate teaching device — real captures don't always land the artifact on the exact frame you expect, so learning to search adjacent frames/sessions is the actual skill being tested.
- Small things like sorting the Credentials tab by username before scanning it saved real time versus filtering blind.
