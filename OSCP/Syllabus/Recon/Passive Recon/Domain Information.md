---
tags:
  - HTB-Academy
---
Start by examing company websites and other materials and getting an idea of what products and services they manage.

# SSL Certificates 

SSL certificates can provide us lots of information about a domain because presumably it will be used for subdomains as well. Here are some tools we can use to examine them.
## [crt.sh](https://crt.sh/)
* Based off [Certificate Transparency](https://en.wikipedia.org/wiki/Certificate_Transparency) A internet system used to validate information about certificates by providing verifiable information about the company.
* Provides Large list of Subdomains 
* Exportable to JSON

## Parsing [crt.sh](https://crt.sh/)
We can use  Its web api to get data about our target, in this case `inlanefreight`
```shell
becker63@htb[/htb]$ curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .

[
  {
    "issuer_ca_id": 23451835427,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "matomo.inlanefreight.com",
    "name_value": "matomo.inlanefreight.com",
    "id": 50815783237226155,
    "entry_timestamp": "2021-08-21T06:00:17.173",
    "not_before": "2021-08-21T05:00:16",
    "not_after": "2021-11-19T05:00:15",
    "serial_number": "03abe9017d6de5eda90"
  }
  ]
```
HTB then does some shell magic to parse the JSON for all the subdomains, but I would just use python or js.
```shell
becker63@htb[/htb]$ curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u

account.ttn.inlanefreight.com
blog.inlanefreight.com
bots.inlanefreight.com
console.ttn.inlanefreight.com
ct.inlanefreight.com
data.ttn.inlanefreight.com
*.inlanefreight.com
inlanefreight.com
integrations.ttn.inlanefreight.com
iot.inlanefreight.com
mails.inlanefreight.com 
...
```
Then HTB does some more shell magic to find all the hosts that are accessable in this list from the internet 
```shell
becker63@htb[/htb]$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done

blog.inlanefreight.com 10.129.24.93
inlanefreight.com 10.129.27.33
matomo.inlanefreight.com 10.129.127.22
www.inlanefreight.com 10.129.127.33
s3-website-us-west-2.amazonaws.com 10.129.95.250
```
## Getting port information about IPs with shodan

Rather than scanning manually ourselves for more information about the services we find we can use shodan in the shell to get information about running services.

```shell
becker63@htb[/htb]$ for i in $(cat ip-addresses.txt);do shodan host $i;done

10.129.24.93
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-09-01T09:02:11.370085
Number of open ports:    2

Ports:
     80/tcp nginx 
    443/tcp nginx 
	
10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-08-30T22:25:31.572717
Number of open ports:    3

Ports:
     22/tcp OpenSSH (7.6p1 Ubuntu-4ubuntu0.3)
     80/tcp nginx 
    443/tcp nginx 
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, -TLSv1.3, TLSv1.2
        |-- Diffie-Hellman Parameters:
                Bits:          2048
                Generator:     2
```
# DNS records 

We can also of course just `dig` our way to the target. Here is some information that HTB says we will find by just doing DNS lookups.

- `A` records: We recognize the IP addresses that point to a specific (sub)domain through the A record. Here we only see one that we already know.
    
- `MX` records: The mail server records show us which mail server is responsible for managing the emails for the company. Since this is handled by google in our case, we should note this and skip it for now.
    
- `NS` records: These kinds of records show which name servers are used to resolve the FQDN to IP addresses. Most hosting providers use their own name servers, making it easier to identify the hosting provider.
    
- `TXT` records: this type of record often contains verification keys for different third-party providers and other security aspects of DNS, such as [SPF](https://datatracker.ietf.org/doc/html/rfc7208), [DMARC](https://datatracker.ietf.org/doc/html/rfc7489), and [DKIM](https://datatracker.ietf.org/doc/html/rfc6376), which are responsible for verifying and confirming the origin of the emails sent. Here we can already see some valuable information if we look closer at the results.

Lets examine the `TXT` record HTB wants us to look at. Thankfully they formatted it into a nice table for us like always.

|   |   |   |
|---|---|---|
|[Atlassian](https://www.atlassian.com/)|[Google Gmail](https://www.google.com/gmail/)|[LogMeIn](https://www.logmein.com/)|
|[Mailgun](https://www.mailgun.com/)|[Outlook](https://outlook.live.com/owa/)|[INWX](https://www.inwx.com/en) ID/Username|
|10.129.24.8|10.129.27.2|10.72.82.106|
WE now have much more information about the services the company uses.
