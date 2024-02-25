---
tags:
  - HTB-Academy
  - web-guide
---
# Filtered ports

*"When a port is shown as filtered, it can have several reasons. In most cases, firewalls have certain rules set to handle specific connections."*

*"When a packet gets dropped, `Nmap` receives no response from our target, and by default, the retry rate (`--max-retries`) is set to 1. This means `Nmap` will resend the request to the target port to determine if the previous packet was not accidentally mishandled." -HTB*

The packets can either be `dropped`, or `rejected`. A `drop` is silent and does not come with a message about why it was rejected. A `rejected`is loud and sometimes comes with a message about why it was rejected.
## Rejected flags
- Net Unreachable
- Net Prohibited
- Host Unreachable
- Host Prohibited
- Port Unreachable
- Proto Unreachable

# Scanning for Stateful vs stateless firewalls with`ACK`

```embed
title: "Determining Firewall Rules | Nmap Network Scanning"
image: "https://nmap.org/images/sitelogo.png"
description: "The first step toward bypassing firewall rules is to understand them. Where possible, Nmap distinguishes between ports that are reachable but closed, and those that are actively filtered. An effective technique is to start with a normal SYN port scan, then move on to more exotic techniques such as`ACK`scan and IP ID sequencing to gain a better understanding of the network."
url: "https://nmap.org/book/determining-firewall-rules.html#defeating-firewalls-ids-ackscan"
```

Because of the limited amount of information an `ACK` scan has as we saw in [[Scan Types]] (Remmeber it can really only provide `filtered`) `ACK` scans of networks are not as useful in providing lots of information about firewalled ports, l unlike `SYN` scans which get blocked, providing us more information.

*"Unlike outgoing connections, all connection attempts (with the `SYN` flag) from external networks are usually blocked by firewalls." -HTB*

`ACK` however is useful for determining whether or not a firewall is stateful or stateless. Because it has the ability to sneak through stateless firewalls that do not want external internet users to connect inbound to your network.

*"Blocking incoming SYN packets (without the`ACK`bit set) is an easy way to do this, but it still allows any`ACK`packets through. Blocking those`ACK`packets is more difficult, because they do not tell which side started the connection... To block unsolicited`ACK`packets firewalls must statefully watch every established connection to determine whether a given`ACK`is appropriate" -nmap.org*

Because `ACK` is so sneaky we can also use it to **bypass** some stateless firewalls

# Detecting IPS systems 

Essentially the only real thing we can do to detect these is to delibrately do something loud like a scan in order to detect them. Nmap has scripts that will do this for us.

# Evading IPS systems
## Decoys

```embed
title: "Firewall/IDS Evasion and Spoofing | Nmap Network Scanning"
image: "https://nmap.org/images/sitelogo.png"
description: "Many Internet pioneers envisioned a global open network with a universal IP address space allowing virtual connections between any two nodes. This allows hosts to act as true peers, serving and retrieving information from each other. People could access all of their home systems from work, changing the climate control settings or unlocking the doors for early guests. This vision of universal connectivity has been stifled by address space shortages and security concerns. In the early 1990s, organizations began deploying firewalls for the express purpose of reducing connectivity. Huge networks were cordoned off from the unfiltered Internet by application proxies, network address translation, and packet filters. The unrestricted flow of information gave way to tight regulation of approved communication channels and the content that passes over them."
url: "https://nmap.org/book/man-bypass-firewalls-ids.html "
```

Nmap has a script built in called the Decoy scanning method (`-D`). It sets your source IP Address to a random IP for certain specific scans.

*"Thus their IDS might report 5–10 port scans from unique IP addresses, but they won't know which IP was scanning them and which were innocent decoys. While this can be defeated through router path tracing, response-dropping, and other active mechanisms, it is generally an effective technique for hiding your IP address." -nmap.org*

*"With this method, Nmap generates various random IP addresses inserted into the IP header to disguise the origin of the packet sent. With this method, we can generate random (`RND`) a specific number (for example: `5`) of IP addresses separated by a colon (`:`). Our real IP address is then randomly placed between the generated IP addresses." -HTB*

*"Another critical point is that the decoys must be alive. Otherwise, the service on the target may be unreachable due to SYN-flooding security mechanisms." -HTB*

*"The spoofed packets are often filtered out by ISPs and routers, even though they come from the same network range. Therefore, we can also specify our VPS servers' IP addresses and use them in combination with "`IP ID`" manipulation in the IP headers to scan the target." -HTB*

```shell
becker63@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sS -Pn -n -D RND:5

```
## Spoofing your source address 

You can change your nmap source address to make it look like it came from a resonable subnet By using the (`-S`) option.
```shell
becker63@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
```
## DNS proxying 

*"However, `Nmap` still gives us a way to specify DNS servers ourselves (`--dns-server <ns>,<ns>`). This method could be fundamental to us if we are in a demilitarized zone (`DMZ`). The company's DNS servers are usually more trusted than those from the Internet. So, for example, we could use them to interact with the hosts of the internal network." -HTB*

## Spoofing

Some services need to be allowed through the firewall in order for them to function, we can exploit that.

*"An administrator will set up a shiny new firewall, only to be flooded with complaints from ungrateful users whose applications stopped working. In particular, DNS may be broken because the UDP DNS replies from external servers can no longer enter the network. FTP is another common example. In active FTP transfers, the remote server tries to establish a connection back to the client to transfer the requested file."*

**Heres what filtered not spoofed traffic looks like (Behind a firewall)**
```shell
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2
```

**And heres what unfiltered spoofed traffic would look like (Behind a firewall)**
```shell
becker63@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

# Unfiltered output
PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
```
