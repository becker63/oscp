---
tags:
  - HTB-Academy
---
# <mark class="hltr-pink">Scan IP List</mark>
*scan targets in a list that looks like this..*
```hosts.lst
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```
## Nmap command
```shell
nmap -sn -oA -iL hosts.lst 
              /\
```
# <mark class="hltr-orange">Scan multiple IPs</mark> 
*how to tell nmap to scan multiple ips*
## Specify multiple IP addresses 
```shell
nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20
```
## <mark class="hltr-green">IP range</mark> 
*How to tell nmap to scan a subnet or range*
## Scan range 
```shell
nmap -sn -oA tnet 10.129.2.18-20
                             /\
```
## Scan subnet 
```shell
nmap -sP 192.168.1.0/24
```
## Scan subnet + exclude specific address 
```shell
nmap 192.168.1.0/24 --exclude 192.168.1.10
```
