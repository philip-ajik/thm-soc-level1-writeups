# Day 28: Network Traffic Basics

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ✅ Completed

---

## 📌 Overview

Network Traffic Analysis (NTA) isn't just "using Wireshark" — it's correlating logs, doing deep packet inspection, and pulling network flow statistics together with a specific goal in mind: full visibility into what's moving in and out of the network, and knowing what's normal versus what deviates from baseline.

The room opened with a scenario that makes the case for NTA well: an SOC analyst gets an alert for an unusual volume of DNS queries from a host, all hitting the same top-level domain with different subdomains each time. DNS logs alone (query, subdomain/TLD, host IP, destination IP, timestamp) aren't enough to draw a conclusion — you need to inspect the actual content of the DNS traffic, because that's where a threat actor could be smuggling C2 instructions through TXT records. Logs tell you *that* a query happened; only the packet content tells you *what was actually said*.

That maps onto the TCP/IP stack directly — each layer only logs a fragment of what it actually carries:
- **Application** — headers + payload (e.g. an HTTP GET/response). Most proxies log the headers but not the payload, so a malicious ZIP's actual content is invisible in the logs.
- **Transport** — TCP/UDP headers. Logs usually keep source/destination port and flags but drop sequence numbers, which is exactly what you'd need to catch session hijacking.
- **Internet** — source/destination IP and TTL are typically logged, but fragment offset and total length (needed to catch fragmentation attacks like overlapping byte ranges) usually aren't.
- **Link** — MAC addressing. Logs won't show a MAC address showing up across multiple interfaces or a flood of gratuitous ARP replies, which is what ARP poisoning looks like in practice.

Traffic sources split into **intermediary** (firewalls, switches, routers, proxies — they pass traffic more than they generate it) and **endpoint** (servers, hosts, IoT, anything traffic actually originates/ends at). Flows split into **North-South** (crosses the firewall, LAN↔WAN — HTTPS, DNS, SSH, VPN, SMTP, RDP) and **East-West** (stays inside the LAN — Kerberos/LDAP auth, SMB file shares, internal DNS, DHCP, database/API traffic, backup replication, SNMP/Syslog/NetFlow).

To actually observe traffic, you've got three options: **logs** (vendor-dependent, rarely a full packet), **full packet capture** (via a physical network TAP — link-layer only, no MAC/IP needed, near-zero performance hit — or port mirroring/SPAN, which is software-based and can add overhead on busy ports), and **network statistics** (NetFlow or its vendor-neutral successor IPFIX — metadata about flows rather than packet contents, useful for spotting C2 traffic, exfiltration, and lateral movement without the storage cost of full packet capture).

---

## 🛠️ Tools Used

- TryHackMe's static "Traffic Analysis Basics" exercise site — drag-and-drop network TAP placement + built-in packet viewer (no separate Wireshark instance used for this room)

---

## 🪜 Steps Followed

**1. Read the exercise instructions**
Objective: place the network tap in the most efficient spot to actually capture the traffic in question — the wrong location just won't show it.

![Exercise instructions](screenshots/01-exercise-instructions.png)

**2. Started Scenario 1/2 — Malicious PS Download**
A user clicked a phishing link, triggering an HTTP request for a malicious PowerShell file. Task: find the right tap placement to capture that web traffic and recover the flag from the HTTP response.

![Scenario 1 start — Malicious PS Download](screenshots/02-scenario1-malicious-ps-download-start.png)

**3. Looked at the full network diagram**
Reviewed the layout before placing the tap — workstation, firewall, router, switches, mail/DNS servers, and endpoint hosts.

![Network diagram with tap slots, running](screenshots/03-network-diagram-running-tap-slots.png)

**4. Placed the tap on the Web Proxy**
Correct on this attempt — the web proxy handles all HTTP(S) requests leaving and entering the network, so it's the one point that sees all web traffic.

![Tap correctly placed on Web Proxy](screenshots/04-scenario1-tap-correct-web-proxy.png)

**5. Found the HTTP GET request in the packet capture**
Captured the client's GET request for the file.

![Captured HTTP GET request](screenshots/05-scenario1-packets-http-get-request.png)

**6. Found the flag in the HTTP response**
The server's 200 response body contained the flag: `THM{FoundTheMalware}`.

![HTTP response with flag](screenshots/06-scenario1-http-response-flag.png)

**7. Started Scenario 2/2 — DNS Infiltration**
A workstation was compromised and C2 instructions were being smuggled in via DNS TXT records. Task: find the right tap placement for DNS traffic and recover the flag.

![Scenario 2 start — DNS Infiltration](screenshots/07-scenario2-dns-infiltration-start.png)

**8. First tap placement attempt was wrong**
Placed it on a switch — got told this location can only show web traffic entering/exiting the network, not DNS traffic specifically.

![Wrong tap placement](screenshots/08-scenario2-tap-wrong-placement.png)

**9. Placed the tap on the DNS server**
Correct — the DNS server handles all external DNS queries and replies on behalf of the host, so all external DNS traffic passes through it.

![Tap correctly placed on DNS server](screenshots/09-scenario2-tap-correct-dns-server.png)

**10. Found the flag in the DNS TXT record response**
Had to use the hint for this one — "C2 commands are often infiltrated via TXT records" — before spotting the DNS response containing `THM{C2CommandFound}`.

![DNS TXT record response with flag](screenshots/10-scenario2-dns-txt-record-flag.png)

---

## 🔍 Key Findings

- **Flag 1:** `THM{FoundTheMalware}` — recovered from the HTTP response body after tapping the Web Proxy in Scenario 1.
- **Flag 2:** `THM{C2CommandFound}` — recovered from a DNS TXT record response after tapping the DNS server in Scenario 2; needed the hint to get there.
- Tap placement isn't one-size-fits-all — the correct spot depends entirely on which traffic type you're after. A switch showed web traffic fine but wasn't sufficient for DNS visibility; only the actual DNS server had full visibility into that flow.
- Logs are a starting point, not the finish line — both flags here lived in the *content* of a packet (an HTTP response body, a DNS TXT record), which is exactly the kind of data that standard logging on these devices wouldn't have captured.

---

## 💡 Lessons Learned

- Getting the wrong-placement message on Scenario 2 was actually useful — it made the North-South vs. specific-device distinction concrete instead of just theoretical (a switch sees traffic passing through it, but that's not the same as being the authoritative source for a given protocol like DNS).
- Needing the hint for the second flag was a good reminder that "DNS tunneling" isn't abstract — TXT records are a real, standard-looking DNS response type that C2 traffic hides inside, and you have to actually go looking at the reply content, not just the query, to catch it.
- This connects directly to Day 27's alert #1034 in the Phishing Unfolding sim — `nslookup.exe` piping a base64-looking string to an external domain was exactly this kind of DNS-based C2/exfiltration pattern, just seen from the process side (Sysmon) instead of the packet-capture side. Nice to have both angles on the same technique back to back.

