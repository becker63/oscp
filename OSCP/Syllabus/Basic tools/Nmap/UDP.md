---
tags:
  - HTB-Academy
---
* ⚠ Some system administrators sometimes forget to filter the UDP ports in addition to the TCP ones.
* Make sure to scan UDP along with Tcp 
* ⚠ we cannot determine if the UDP packet has arrived at all or not. 
	* If the UDP port is `open`, we only get a response if the application is configured to do so.
* If the UDP port does not respond, nmap sets the port state to.. `open|filtered` 
## UDP scan example
```shell
nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
```
In the output the only reason we can get is `udp-response`
```output
PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
```

