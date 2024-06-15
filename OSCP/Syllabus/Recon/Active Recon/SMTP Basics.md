---
tags:
  - HTB-Academy
---
# What is smtp?
* The protocol used to send emails. SMTP servers are sent emails from clients through [[IMAP or POP3 Basics|IMAP/POP3]] and then the server sends those emails out to where they need to go. You can effectively most of the time think of a SMTP server as a client.
* Ports 
	* port `25`.
	* Newer servers use port `587`
		* this port is used to receive mail from authenticated servers 
* Security 
	* smtp servers authenticate correct users using ESMTP/SMTP-Auth
		* This helps prevent spam
	* SMTP by defualt sends all commands data and auth commands in plain text 
		* Newer secure versions use SSL/TLS to prevent this 
		* These SSL/TLS versions might use an extra port like `465` to do secure connnections
* `MUA` + `MTA` + `MSA`
	* `Mail User Agent` (`MUA`) 
		* The name of the smtp client
	* `Mail Transfer Agent` (`MTA`)
		* Part of the `MUA` which actually send the emails and checks for spam. 
		* Also called relay servers 
	*  `Mail Submission Agent` (`MSA`) 
		* Part of the `MUA` which checks the origin/valitity of the emails it was sent 

|Client (`MUA`)|`➞`|Submission Agent (`MSA`)|`➞`|Open Relay (`MTA`)|`➞`|Mail Delivery Agent (`MDA`)|`➞`|Mailbox (`POP3`/`IMAP`)|
|---|---|---|---|---|---|---|---|---|
* SMTP relays
	* Sometimes email servers dont authenticate the senders of emails 
	* Attackers can exploit this fact to send very large amounts of spam with fake sending addresses (Spoofing)
		* There are ways to combat this problem with the protocol 
			* [DomainKeys](http://dkim.org/) (`DKIM`)
			* [Sender Policy Framework](https://dmarcian.com/what-is-spf/) (`SPF`)
			* `Extended SMTP` (`ESMTP`)
				* With its `AUTH PLAIN` command

# SMTP Postfix config 

```shell
becker63@htb[/htb]$ cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"

smtpd_banner = ESMTP Server 
biff = no
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
myhostname = mail1.inlanefreight.htb
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
smtp_generic_maps = hash:/etc/postfix/generic
mydestination = $myhostname, localhost 
masquerade_domains = $myhostname
mynetworks = 127.0.0.0/8 10.129.0.0/16
mailbox_size_limit = 0
recipient_delimiter = +
smtp_bind_address = 0.0.0.0
inet_protocols = ipv4
smtpd_helo_restrictions = reject_invalid_hostname
home_mailbox = /home/postfix
```

|**Command**|**Description**|
|---|---|
|`AUTH PLAIN`|AUTH is a service extension used to authenticate the client.|
|`HELO`|The client logs in with its computer name and thus starts the session.|
|`MAIL FROM`|The client names the email sender.|
|`RCPT TO`|The client names the email recipient.|
|`DATA`|The client initiates the transmission of the email.|
|`RSET`|The client aborts the initiated transmission but keeps the connection between client and server.|
|`VRFY`|The client checks if a mailbox is available for message transfer.|
|`EXPN`|The client also checks if a mailbox is available for messaging with this command.|
|`NOOP`|The client requests a response from the server to prevent disconnection due to time-out.|
|`QUIT`|The client terminates the session.|
## Dangerous settings

*"Often, administrators have no overview of which IP ranges they have to allow. This results in a misconfiguration of the SMTP server that we will still often find in external and internal penetration tests. Therefore, they allow all IP addresses not to cause errors in the email traffic and thus not to disturb or unintentionally interrupt the communication with potential and current customers." -HTB*

Heres what the setting you should look for when your trying to determine if something is an open relay 

```shell
mynetworks = 0.0.0.0/0
```

# Connecting to the service with telnet

## HELO/EHLO
In order to initalize the session you must send a HELO or EHLO (weird) command
```shell
becker63@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 


HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb


EHLO mail1 # Sending the EHLO command 

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

## Send an email 

```shell
becker63@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


EHLO EHLO EHLO inlanefreight.htb

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING


MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.

250 2.0.0 Ok: queued as 6E1CF1681AB


QUIT

221 2.0.0 Bye
Connection closed by foreign host.
```

