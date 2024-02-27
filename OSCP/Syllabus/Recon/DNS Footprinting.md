---
tags:
  - web-guide
---
```embed
title: "53 - Pentesting DNS - HackTricks"
image: "https://2783428383-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/collections%2FmuMguNrsRx2mNyNqEox4%2Ficon%2F1qCJ0VIDlWcvGSecYCDq%2Ffondo.png?alt=media&token=1e721267-450f-43f3-861b-6c4f93278e93"
description: "Other ways to support HackTricks:"
url: "https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns"
```
# Things to look for

When enumerating subdomains make sure to look for eye catching domains like "Internal" or other interesting things, and then get more information about those. Thats what I ran into trouble with on HTB.

# Doing A zone transfer 

```shell
dig axfr @<DNS_IP> #Try zone transfer without domain
dig axfr @<DNS_IP> <DOMAIN> #Try zone transfer guessing the domain
fierce --domain <DOMAIN> --dns-servers <DNS_IP> #Will try toperform a zone transfer against every authoritative name server and if this doesn'twork, will launch a dictionary attack
```

Here they use the [Fierce](https://www.kali.org/tools/fierce/) hacking tool, but you can also use [DNSenum](https://github.com/fwaeytens/dnsenum). Both include zone transfer functions. 

# Tools 
* ## Fierce 
	* https://www.kali.org/tools/fierce/
	* Meant to be used before nmap or other scanners 
	* lightweight and fast
	* Not as many functions as DnsEnum
* ## DnsEnum 
	* https://www.kali.org/tools/dnsenum/
	* Lots of different scans ran for you 
		- Get the host’s addresses (A record).
		- Get the namservers (threaded).
		- Get the MX record (threaded).
		- Perform axfr queries on nameservers and get BIND versions(threaded).
		- Get extra names and subdomains via google scraping (google query = “allinurl: -www site:domain”).
		- Brute force subdomains from file, can also perform recursion on subdomain that have NS records (all threaded).
		- Calculate C class domain network ranges and perform whois queries on them (threaded).
		- Perform reverse lookups on netranges (C class or/and whois netranges) (threaded).
		- Write to domain_ips.txt file ip-blocks.
	- Slower but more comprehensive  

# Brute force/Dictionary  
## Subdomain bruteforce
```shell
dnsenum --dnsserver <IP_DNS> --enum -p 0 -s 0 -o subdomains.txt -f subdomains-1000.txt <DOMAIN>
dnsrecon -D subdomains-1000.txt -d <DOMAIN> -n <IP_DNS>
dnscan -d <domain> -r -w subdomains-1000.txt #Bruteforce subdomains in recursive way, https://github.com/rbsec/dnscan
```
## Reverse dns lookup Brute force
*Remember, a reverse lookup resolves a IP => Domain* 
```shell
dnsrecon -r 127.0.0.0/24 -n <IP_DNS>  #DNS reverse of all of the addresses
dnsrecon -r 127.0.1.0/24 -n <IP_DNS>  #DNS reverse of all of the addresses
dnsrecon -r <IP_DNS>/24 -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d active.htb -a -n <IP_DNS> #Zone transfer
dig -x [address] # manual lookup
```
## Dictionary subdomain search 

```shell
for sub in $(cat <WORDLIST>);do dig $sub.<DOMAIN> @<DNS_IP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

dnsenum --dnsserver <DNS_IP> --enum -p 0 -s 0 -o subdomains.txt -f <WORDLIST> <DOMAIN>
```

# Useful metasploit/nmap scripts 

## Metasploit 

```shell
auxiliary/gather/enum_dns #Perform enumeration actions
```
## Nmap

*Basic enumereation*
```shell
#Perform enumeration actions
nmap -n --script "(default and *dns*) or fcrdns or dns-srv-enum or dns-random-txid or dns-random-srcport" <IP>
```

*Query [[DNS Basics#Whats DNSEC?|DNSEC]] info* 
```shell
 #Query paypal subdomains to ns3.isc-sns.info
 nmap -sSU -p53 --script dns-nsec-enum --script-args dns-nsec-enum.domains=paypal.com ns3.isc-sns.info
```

#  DNS Queries 

## ANY record
*You absolutely need to run this, it returns all the records the server is willing to disclose*
```shell
dig any victim.com @<DNS_IP>
```

## CHAOS Query 
*Some servers will return the DNS servers version and the servers 'banner' if you query this*
```shell
dig version.bind CHAOS TXT @DNS
```

## [[Active Directory]] Query
```shell
dig -t _gc._tcp.lab.domain.com
dig -t _ldap._tcp.lab.domain.com
dig -t _kerberos._tcp.lab.domain.com
dig -t _kpasswd._tcp.lab.domain.com

nslookup -type=srv _kerberos._tcp.<CLIENT_DOMAIN>
nslookup -type=srv _kerberos._tcp.domain.com

nmap --script dns-srv-enum --script-args "dns-srv-enum.domain='domain.com'"
```

# IPV6

*Brute force using "AAAA" requests to gather IPv6 of the subdomains.*
```shell
dnsdict6 -s -t <domain>
```

*Bruteforce reverse DNS in using IPv6 addresses*
```shell
dnsrevenum6 pri.authdns.ripe.net 2001:67c:2e8::/48 #Will use the dns pri.authdns.ripe.net
```

# Local Host dns config file locations 

```shell
host.conf

/etc/resolv.conf
/etc/bind/named.conf
/etc/bind/named.conf.local
/etc/bind/named.conf.options
/etc/bind/named.conf.log
/etc/bind/*
```
