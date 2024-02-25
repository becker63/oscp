---
tags:
  - web-guide
---
```embed
title: "NMAP Flag Guide: What They Are, When to Use Them"
image: "https://images.ctfassets.net/aoyx73g9h2pg/313OdR8DXlnnHjgfRXgSlW/fd963be8b256930d42d72a3c3d4194cb/1chu_WJtfj99XYq4ktaIOeNwC7GdgMyS5_3-Featured-1024x572.jpg?fm=jpg&w=1024"
description: "If penetration testing and a career in cybersecurity interests you, learn how to use Nmap and its flags. Oh, and you’ll need to understand Nmap to pass CompTIA’s PenTest+ exam."
url: "https://www.cbtnuggets.com/blog/certifications/security/nmap-flags-what-they-are-when-to-use-them"
```
# Nmap syntax 
```shell-session
becker63@htb[/htb]$ nmap <scan types> <options> <target>
```
## <mark class="hltr-orange">Scan Type Flags </mark>
`Options for connections`
* ## `-sS`
	* ***Tcp Syn*** 
	* *determines the state of the port without fully connecting to the target system*
* ## `-sT`
	* ***Tcp Connect***
	* *Open a connection to tcp ports*
* ## `-sA`
	* ***Tcp Ack***
	* *Helps determine if a tcp port is filtered/firewalled*
* ## `-sU`
	* ***UDP***
	* *Perform a UDP connection*

## <mark class="hltr-purple">Host Discovery Options</mark>
`Options for searching for hosts`
* ## `-Pn`
	* ***disable ping***
	* *disables pinging the target and only scans for open ports, avoiding detection*
* ## `-sn`
	* ***only host discovery***
	* *Only looks for online hosts without port scanning*
* ## `-PR`
	* ***ARP discovery***
	* *looks for local machines and local mac address linkings*
* ## `-n`
	* ***Do not dns resolve***
	* *Do not dns resolve, dns can take a while so its useful for quick checks*
* ## `--disable-arp-ping`
	* ***disable arp ping***
	* *Disables ARP ping.*
## <mark class="hltr-pink">Port Specification Options</mark>
`Options for scanning through ports`
* ## `-p 1-9000`
	* ***specific port/range***
	* *scan for specific port or port range*
* ## `-p-`
	* ***all ports***
	* *scan all the ports on the machine*
* ## `-F`
	* ***fast  scan***
	* *performs a fast port scan*
* ## `--top-ports=10`
	* ***top ports***
	* *scans top ten (or other specified number) ports used on computers*
## <mark class="hltr-cyan">Service Options</mark>
`Extra service options nmap has`
* ## `-A`
	* ***Aggressive Detection/All Mode**
	* *Runs all of the below operations at once*
* ## `-O`
	* ***Os detection mode***
	* *detects what operating system the machine is running*
* ## `-sV`
	* ***Service versions***
	* *tells you what version each service is*
* ## `--packet-trace`
	* ***packet trace***
	* *shows all packets being sent and received*
* ## `--reason`
	* ***reason***
	* *tells us the reason a packet might be closed or up* 
	* *⚠NOTE: This is not very useful with UDP*
## <mark class="hltr-blue">Output Format Options</mark>
`options for formatting output for a file/pipe`
* ## `-oN`
	* ***Normal***
	* *Just normal file output*
* ## `-oX`
	* ***XML***
	* *xml output format*
* ## `-oG`
	* ***Grep***
	* *Outputs to a format that is easily greppable*
* ## `-oA`
	* ***All***
	* *does all of the output formats at once, useful for reports*
* ## `--stats-every=5s`
	* *Shows the progress of the scan every 5 seconds.*
* ## `-v; -vv`
	* ***verbosity level***
	* *will show us the open ports directly when `Nmap` detects them.*