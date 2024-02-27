---
tags:
  - web-guide
---
I dont think hack the box did as good a job covering dns, so I just took a look at hacktricks and did some research on zone transfers and DNSEC.

# Whats DNSEC?

https://www.cloudflare.com/dns/dnssec/how-dnssec-works/
```embed
title: "How DNSSEC Works"
image: "https://cf-assets.www.cloudflare.com/slt3lc6tev37/3zq5CxyzG0OWC2yWEaWmqG/c1639744ab7529c3aaa514ab436e4ed1/dnssec-logo.svg"
description: "DNSSEC helps The domain name system (DNS) which the phone book of the Internet to be secure, Find out how it works here."
url: "https://www.cloudflare.com/dns/dnssec/how-dnssec-works/"
```

DNSEC is a part of dns that prevents users from doing dns poisoning by adding public key cryptography to the lookups. Each record in a dns zone has an associated Signature that can be verified 

DNSEC gets very complicated and probably (Hopefully) wont be touched on in the oscp. The only thing that is relevant to us about it is the information older versions of it can leak.

DNSEC Adds a new record called a Explicit Denial of Existence or a **NSEC** record. They both exist to tell users that a ip address of a domain doesnt exist. 

*"NSEC works by returning the “next secure” record. For example, consider a name server that defines AAAA records for api, blog, and www. If you request a record for store, it would return an NSEC record containing www, meaning there’s no AAAA records between store and www when the records are sorted alphabetically. This effectively tells you that store doesn’t exist." -Cloudfare*

We can use this fact to enumerate domains with something like nmaps `dns-nsec-enum` script. 

This enumeration method only works on servers that use **NSEC**. The newer version, **NSEC3** will not work. 

# Whats a zone transfer?

```embed
title: "What are DNS zone transfers (AXFR)?"
image: "https://cdn.acunetix.com/wp-content/uploads/2019/07/11110842/doh_featured.png"
description: "DNS zone transfers using the AXFR protocol are the simplest mechanism to replicate DNS records across DNS servers. To avoid the need to edit information on multiple DNS servers, you can edit information on one server and use AXFR to copy information to other servers."
url: "https://www.acunetix.com/blog/articles/dns-zone-transfers-axfr/"
```

When large organizations implement dns they often want to have at least two redudant dns servers because of how critical it is. With a small amount of dns servers IT administrators might be able to edit and manually copy changes made to the dns servers they manage but as networks get larger this manual copying becomes unfeasable. 

Smart admins use a function built into the protocol called **zone transfers**. There are lots of different specific ways you can do a zone transfer but the simplest and most common one is called **AXFR**. 

If An attacker is able to perform an **AXFR** zone transfer on the target machine they could get information about every host on the network that has a network name, massively increasing the amount of attack vectors they  have. 

Typically in order to stop zone transfer exploitation admins create a IP whitelist, which only allows trusted IP addresses to perform them.