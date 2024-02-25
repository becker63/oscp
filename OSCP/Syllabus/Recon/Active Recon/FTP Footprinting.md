---
tags:
  - HTB-Academy
---
Finnally we are doing the fun stuff. Lets enumerate this service.

# [[Nmap]] FTP scripts 

```shell
becker63@htb[/htb]$ find / -type f -name ftp* 2>/dev/null | grep scripts

/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse
```

| Script Name             | Description                                                                                                                                                                                                                                                                                                                                                                                                                    | Categories                                                                                                                                                                                                                                                                                  | Link                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| `ftp-syst`              | Sends FTP SYST and STAT commands and returns the result.                                                                                                                                                                                                                                                                                                                                                                       | _[default](https://nmap.org/nsedoc/categories/default.html)_                [discovery](https://nmap.org/nsedoc/categories/discovery.html)       _[safe](https://nmap.org/nsedoc/categories/safe.html)_                                                                                     | [link](https://nmap.org/nsedoc/scripts/ftp-syst.html)              |
| `ftp-vsftpd-backdoor`   | Tests for the presence of the vsFTPd 2.3.4 backdoor reported on 2011-07-04 (CVE-2011-2523). This script attempts to exploit the backdoor using the innocuous `id` command by default, but that can be changed with the `exploit.cmd` or `ftp-vsftpd-backdoor.cmd` script arguments.                                                                                                                                            | _[exploit](https://nmap.org/nsedoc/categories/exploit.html)_,                 _[intrusive](https://nmap.org/nsedoc/categories/intrusive.html)_,  _[malware](https://nmap.org/nsedoc/categories/malware.html)_,<br/>               _[vuln](https://nmap.org/nsedoc/categories/vuln.html)_    | [link](https://nmap.org/nsedoc/scripts/ftp-vsftpd-backdoor.html)   |
| `ftp-vuln-cve2010-4221` | Checks for a stack-based buffer overflow in the ProFTPD server, version between 1.3.2rc3 and 1.3.3b. By sending a large number of TELNET_IAC escape sequence, the proftpd process miscalculates the buffer length, and a remote attacker will be able to corrupt the stack and execute arbitrary code within the context of the proftpd process (CVE-2010-4221). Authentication is not required to exploit this vulnerability. | _[intrusive](https://nmap.org/nsedoc/categories/intrusive.html)_,<br/>                                  _[vuln](https://nmap.org/nsedoc/categories/vuln.html)_                                                                                                                              | [link](https://nmap.org/nsedoc/scripts/ftp-vuln-cve2010-4221.html) |
| `ftp-proftpd-backdoor`  | Tests for the presence of the ProFTPD 1.3.3c backdoor reported as BID 45150. This script attempts to exploit the backdoor using the innocuous `id` command by default, but that can be changed with the `ftp-proftpd-backdoor.cmd` script argument.                                                                                                                                                                            | _[exploit](https://nmap.org/nsedoc/categories/exploit.html)_,      <br/>         _[intrusive](https://nmap.org/nsedoc/categories/intrusive.html)_,  <br/>_[malware](https://nmap.org/nsedoc/categories/malware.html)_,    <br/>      _[vuln](https://nmap.org/nsedoc/categories/vuln.html)_ | [link](https://nmap.org/nsedoc/scripts/ftp-proftpd-backdoor.html)  |
| `ftp-bounce`            | Checks to see if an FTP server allows port scanning using the FTP bounce method.                                                                                                                                                                                                                                                                                                                                               | _[default](https://nmap.org/nsedoc/categories/default.html)_,<br/> _[safe](https://nmap.org/nsedoc/categories/safe.html)_                                                                                                                                                                   | [link](https://nmap.org/nsedoc/scripts/ftp-bounce.html)            |
| `ftp-libopie`           | Checks if an FTPd is prone to CVE-2010-1938 (OPIE off-by-one stack overflow), a vulnerability discovered by Maksymilian Arciemowicz and Adam "pi3" Zabrocki. See the advisory at [https://nmap.org/r/fbsd-sa-opie](https://nmap.org/r/fbsd-sa-opie). Be advised that, if launched against a vulnerable host, this script will crash the FTPd.                                                                                  | _[vuln](https://nmap.org/nsedoc/categories/vuln.html)_, <br/>_[intrusive](https://nmap.org/nsedoc/categories/intrusive.html)_                                                                                                                                                               | [link](https://nmap.org/nsedoc/scripts/ftp-libopie.html)           |
| `ftp-anon`              | Checks if an FTP server allows anonymous logins.<br><br>If anonymous is allowed, gets a directory listing of the root directory and highlights writeable files.                                                                                                                                                                                                                                                                | _[default](https://nmap.org/nsedoc/categories/default.html)_,<br/> _[auth](https://nmap.org/nsedoc/categories/auth.html)_, <br/> _[safe](https://nmap.org/nsedoc/categories/safe.html)_                                                                                                     | [link](https://nmap.org/nsedoc/scripts/ftp-anon.html)              |
| `ftp-brute`             | Performs brute force password auditing against FTP servers.<br><br>Based on old ftp-brute.nse script by Diman Todorov, Vlatko Kosturjak and Ron Bowes                                                                                                                                                                                                                                                                          | _[intrusive](https://nmap.org/nsedoc/categories/intrusive.html)_,<br/> _[brute](https://nmap.org/nsedoc/categories/brute.html)_                                                                                                                                                             | [link](https://nmap.org/nsedoc/scripts/ftp-brute.html)             |
# Defualt ftp nmap command 
```shell
becker63@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```
## Footprinting with openssl
[[FTP Basics#Connecting to the server with openssl]]
* The ssl connection gives us the **Hostname**
* and an **email address** for the company
* and also the ssl certificate obviously