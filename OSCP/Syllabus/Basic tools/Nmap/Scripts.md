---
tags:
  - HTB-Academy
  - web-guide
---
```embed
title: "NSEDoc Reference Portal: NSE Scripts — Nmap Scripting Engine documentation"
image: "https://nmap.org/images/sitelogo.png"
description: "A list of 604 Nmap scripts and their descriptions. Links to more detailed documentation."
url: "https://nmap.org/nsedoc/scripts/"
```

nmap has two ways to start the lua scripts they use, category's and specific files
## Script file
```shell
becker63@htb[/htb]$ sudo nmap <target> --script <category>
```
## Script category 
```shell
becker63@htb[/htb]$ sudo nmap <target> --script <category>
```

| **Category** | **Description**                                                                                                                         |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `auth`       | Determination of authentication credentials.                                                                                            |
| `broadcast`  | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
| `brute`      | Executes scripts that try to log in to the respective service by brute-forcing with credentials.                                        |
| `default`    | Default scripts executed by using the `-sC` option.                                                                                     |
| `discovery`  | Evaluation of accessible services.                                                                                                      |
| `dos`        | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.              |
| `exploit`    | This category of scripts tries to exploit known vulnerabilities for the scanned port.                                                   |
| `external`   | Scripts that use external services for further processing.                                                                              |
| `fuzzer`     | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.     |
| `intrusive`  | Intrusive scripts that could negatively affect the target system.                                                                       |
| `malware`    | Checks if some malware infects the target system.                                                                                       |
| `safe`       | Defensive scripts that do not perform intrusive and destructive access.                                                                 |
| `version`    | Extension for service detection.                                                                                                        |
| `vuln`       | Identification of specific vulnerabilities.                                                                                             |
