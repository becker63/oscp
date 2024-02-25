---
tags:
  - HTB-Academy
---
Sometimes NMAP is unable to provide precise banner information about a port because its scan does not immediately provide it with enough information to grab the banner. 
```shell
- NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
```
According to HTB if we ever encounter errors like these it might be a good idea to mannually connect to the sever with <mark class="hltr-pink">netcat</mark> and inspect the connection with <mark class="hltr-green">tcpdump</mark>

"*After a successful three-way handshake, the server often sends a banner for identification. This serves to let the client know which service it is working with. At the network level, this happens with a `PSH` flag in the TCP header. However, it can happen that some services do not immediately provide such information. It is also possible to remove or manipulate the banners from the respective services. If we manually connect to the SMTP server using <mark class="hltr-pink">netcat</mark>, grab the banner, and intercept the network traffic using <mark class="hltr-green">tcpdump</mark>, we can see what `Nmap` did not show us.*"

# Starting tcpump
```shell
becker63@htb[/htb]$  nc -nv 10.129.2.28 25

Connection to 10.129.2.28 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```
# Creating a netcat connection
```shell
becker63@htb[/htb]$  nc -nv 10.129.2.28 25

Connection to 10.129.2.28 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```
# Tcpdump - Intercepted Traffic 

*"The first three lines show us the three-way handshake."*

| `[SYN]`     | `18:28:07.128564 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S], <SNIP>`  |
| ----------- | ---------------------------------------------------------------------------- |
| `[SYN-ACK]` | `18:28:07.255151 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.], <SNIP>` |
| `[ACK]`     | `18:28:07.255281 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], <SNIP>`  |

*"After that, the target SMTP server sends us a TCP packet with the <mark class="hltr-cyan">PSH</mark> and <mark class="hltr-blue">ACK</mark> flags, where <mark class="hltr-cyan">PSH</mark> states that the target server is sending data to us and with <mark class="hltr-blue">ACK</mark> simultaneously informs us that all required data has been sent."*

| `[PSH-ACK]` | `18:28:07.319306 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.], <SNIP>` |
| ----------- | ---------------------------------------------------------------------------- |

*"The last TCP packet that we sent confirms the receipt of the data with an <mark class="hltr-blue">ACK</mark>."*

| `[ACK]` | `18:28:07.319426 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], <SNIP>` |
| ------- | --------------------------------------------------------------------------- |
