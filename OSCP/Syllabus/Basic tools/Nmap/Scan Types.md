---
tags:
  - HTB-Academy
  - web-guide
---
https://nmap.org/book/man-port-scanning-techniques.html
```embed
title: "Port Scanning Techniques | Nmap Network Scanning"
image: "https://nmap.org/images/sitelogo.png"
description: "As a novice performing automotive repair, I can struggle for hours trying to fit my rudimentary tools (hammer, duct tape, wrench, etc.) to the task at hand. When I fail miserably and tow my jalopy to a real mechanic, he invariably fishes around in a huge tool chest until pulling out the perfect gizmo which makes the job seem effortless. The art of port scanning is similar. Experts understand the dozens of scan techniques and choose the appropriate one (or combination) for a given task. Inexperienced users and script kiddies, on the other hand, try to solve every problem with the default SYN scan. Since Nmap is free, the only barrier to port scanning mastery is knowledge. That certainly beats the automotive world, where it may take great skill to determine that you need a strut spring compressor, then you still have to pay thousands of dollars for it."
url: "https://nmap.org/book/man-port-scanning-techniques.html"
```
# Connect scan 
* ## `-sT`
* Regarded  as the most "Polite" way to scan a network
* uses full TCP three way handshake
* ⚠ most stealthy option 
	* doesnt leave any connections hanging
	* less likely to be detected by IDS
* can sometimes bypass host firewalls
* *slowest* kind of scan 
# SYN scan 
* ## `-sS`
* fastest kind of scan
* most popular kind of scan 
* reliably tells you whether a port is `open`, `closed`, or `filtered`
* very loud 
# ACK scan 
* ## `-sA`
* ⚠ does not determine port state only responds with `filtered`
* used to determine if a firewall is stateful or stateless
## Protocol scan 
* ## `-sO`
* determines what ip protocols run on the machine 
	* TCP, UDP, ICMP, IGMP, SCTP, etc 
# TCP NULL, FIN, and Xmas scans
* ## `-sN; -sF; -sX`
* can only provide `open|filtered` `closed` and `filtered` port states. 
* can sneak through certain non-stateful firewalls and packet filtering routers 
* ⚠ most modern IDS products can be configured to detect them
* not very reliable because not all IP systems follow the RFC specification these are exploiting to the letter
* ⚠ Only works on NIX* machines