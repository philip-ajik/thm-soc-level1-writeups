# Day 32: What is Networking?

**Path:** SOC Level 1
**Platform:** TryHackMe
**Status:** ✅ Completed

---

## 📌 Overview

This room went back to first principles: what a network actually is, and how devices on one identify themselves and each other.

A network is just things connected — the room's comparison is a friendship circle, but the same idea scales from a city's public transport system to the national power grid to billions of computing devices. **The Internet** is one giant network made of many small networks joined together: the small ones are **private networks**, and the connections between them form the **public network** — the Internet itself. Its first iteration was ARPANET in the late 1960s (funded by the US Defence Department), but the Internet as commonly understood didn't exist until 1989, when Tim Berners-Lee created the World Wide Web and the Internet became a place to store and share information rather than just connect machines.

For devices to communicate in an ordered way, they need to be both **identifying and identifiable** — the room draws a direct parallel to humans having a name (changeable) and fingerprints (permanent). Devices get the same two-tier identity:

- **IP address** — a set of numbers split into four octets, used to identify a host on a network. It can change from device to device, but two devices can't hold the same IP address active at the same time on the same network. Depending on where a device sits, its IP is either **public** (identifies it on the Internet, assigned by an ISP) or **private** (identifies it among other devices on its own local network) — a single device commonly has both at once. IPv4 only supports 2³² addresses (~4.3 billion), which is running out given how many devices now connect to the Internet; **IPv6** solves that with 2¹²⁸ addresses (340+ trillion) and more efficient addressing.
- **MAC address** — a twelve-character hexadecimal number burned into a device's network interface at the factory, split into pairs and separated by colons (e.g. `a4:c3:f0:85:ac:2d`). The first six characters identify the manufacturer; the last six are a unique identifier. Unlike an IP, a MAC address is meant to be permanent — but it *can* be faked, a technique called **spoofing**, where a device pretends to be another device's MAC address. This is a real problem for security designs that trust a MAC address as proof of identity: if a firewall is configured to trust traffic from an administrator's MAC address, and an attacker spoofs that same MAC, the firewall has no way to tell the difference.

**Ping (ICMP)** is the fundamental tool for testing whether a connection between two devices exists and how reliable it is. It works by sending an ICMP echo packet and timing how long the ICMP echo reply takes to come back — available out of the box on Linux and Windows via `ping <IP address or URL>`.

---

## 🛠️ Tools Used

- TryHackMe's browser-based "Free Hotel Wifi" interactive lab (MAC address spoofing simulation)
- TryHackMe's browser-based "Network Ping" terminal simulation

---

## 🪜 Steps Followed

**1. Reviewed the Hotel Wifi lab's starting state.** Bob (MAC `04:9E:44:99:A3:12`) hasn't paid for Wi-Fi, so the router drops his packets straight into the bin. Alice (MAC `00:12:32:2F:33:39`) has paid, so her packets go through to the router fine — the lab's whole premise is that the router is filtering purely on MAC address.

![Hotel Wifi lab — Bob blocked, Alice allowed](screenshots/01-hotel-wifi-bob-blocked-alice-allowed.png)

**2. Spoofed Bob's MAC address to match Alice's.** Changed Bob's MAC address field to `00:12:32:2F:33:39` — identical to Alice's — and hit Request Site again. With both devices now presenting the same MAC, the router let Bob's traffic through too and revealed the flag: `THM{YOU_GOT_ON_TRYHACKME}`.

![MAC spoofing bypasses the filter — flag revealed](screenshots/02-mac-spoofing-flag-revealed.png)

**3. Ran the Network Ping practical.** Used `ping -c 4 8.8.8.8` in the deployed terminal, sending four ICMP echo requests to `8.8.8.8` — all four came back with 0% packet loss and an average round-trip time around 9.4ms, and the terminal printed the flag directly: `THM{I_PINGED_THE_SERVER}`.

![Ping 8.8.8.8 — flag revealed](screenshots/03-ping-8888-flag.png)

---

## 🔍 Key Findings

- **Hotel Wifi lab flag:** `THM{YOU_GOT_ON_TRYHACKME}` — obtained by spoofing Bob's MAC address to match Alice's paid-for MAC address, bypassing the router's MAC-based access filter entirely.
- **Network Ping lab flag:** `THM{I_PINGED_THE_SERVER}` — obtained by pinging `8.8.8.8` with `ping -c 4 8.8.8.8`.
- MAC address filtering (as used by "pay for Wi-Fi" style hotel/cafe networks) is trivially bypassable once you can see another device's MAC address on the same network — it's identity by claim, not identity by proof.

---

## 💡 Lessons Learned

- This room is a clean, practical illustration of exactly the kind of weak trust assumption the docs warned about: a firewall or access filter that trusts a MAC address as identity is only as strong as how hard that MAC address is to observe and copy. On a shared network like hotel Wi-Fi, that's not hard at all.
- It's a good grounding for [[Day 31]]'s ARP/MITM work — MAC spoofing here is the same underlying weakness (MAC addresses carry no built-in authentication) that made the ARP poisoning attack in that room possible in the first place, just applied to a different filtering mechanism.
- Small but useful reminder: IPv4's address exhaustion (2³² addresses against tens of billions of connected devices) is the practical reason IPv6 and private/public IP separation exist at all — not just a version-number bump.
