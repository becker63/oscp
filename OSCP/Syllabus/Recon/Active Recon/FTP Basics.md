---
tags:
  - HTB-Academy
---
# <mark class="hltr-pink">FTP Protocol </mark>
*A file transfer protocol*
* ## FTP uses two ports 
	* 21
		* The 'control' channel 
		* the client sends commands 
		* the server sends command status codes 
	* 20
		* the 'data' channel 
		* used exclusively to send files
		* watches for errors on this ports 
* ## Active Vs Passive FTP
	* Active 
		* Client tells server which client-side port the server can transmit its responses to
		* Can fail because firewalls can block the port the client tells the server to use 
	* Passive 
		* Firewall chooses what port it can send data to
		* Doesnt break due to firewalls
* ## FTP return codes 
	* Some ftp client commands will not necessarily  be implemented on the server
	* In order to talk to the client FTP uses return codes 
		* Here is a large [list](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes) of them 
* ## FTP Authentication 
	* FTP is cleartext
	* There is almost always some sort of authentication running on the server 
		* Otherwise The server configures an `anonymous` ftp role 
			* this role usually has less access to the serve 
# <mark class="hltr-red">TFTP Protocol </mark>
*Trivial File Transfer Protocol*
* ## No authentication 
	* TFTP does not suppot any form of authentication/logging in
* ## UDP
	* TFTP uses UDP for some reason 
	* Scans for it must use UDP mode 
* ## TFTP Commands 
	* Note that TFTP Cannot list Directorys

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits tftp.                                                                                                                            |
| `status`     | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |
|              |                                                                                                                                        |
# [vsFTPd](https://security.appspot.com/vsftpd.html) The most common linux FTP server
## Defualt Config 
	* Found in `/etc/vsftpd.conf`
	* /etc/ftpusers
		* Used to deny certain users access to the service 
		* Just a newline delimited list of usernames

| **Setting**                                                   | **Description**                                                                                          |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `listen=NO`                                                   | Run from inetd or as a standalone daemon?                                                                |
| `listen_ipv6=YES`                                             | Listen on IPv6 ?                                                                                         |
| `anonymous_enable=NO`                                         | Enable Anonymous access?                                                                                 |
| `local_enable=YES`                                            | Allow local users to login?                                                                              |
| `dirmessage_enable=YES`                                       | Display active directory messages when users go into certain directories?                                |
| `use_localtime=YES`                                           | Use local time?                                                                                          |
| `xferlog_enable=YES`                                          | Activate logging of uploads/downloads?                                                                   |
| `connect_from_port_20=YES`                                    | Connect from port 20?                                                                                    |
| `secure_chroot_dir=/var/run/vsftpd/empty`                     | Name of an empty directory                                                                               |
| `pam_service_name=vsftpd`                                     | This string is the name of the PAM service vsftpd will use.                                              |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`          | The last three options specify the location of the RSA certificate to use for SSL encrypted connections. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` |                                                                                                          |
| `ssl_enable=NO`                                               |                                                                                                          |

# Dangerous settings 

| **Setting**                    | **Description**                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| `anonymous_enable=YES`         | Allowing anonymous login?                                                          |
| `anon_upload_enable=YES`       | Allowing anonymous to upload files?                                                |
| `anon_mkdir_write_enable=YES`  | Allowing anonymous to create new directories?                                      |
| `no_anon_password=YES`         | Do not ask anonymous for password?                                                 |
| `anon_root=/home/username/ftp` | Directory for anonymous.                                                           |
| `write_enable=YES`             | Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE? |
## Hiding UID and GUID 

If we were to list a directory in ftp that could provide us information about the groups and users on the system. Note the `1002` In the command output.
```ftp
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```
We can turn this off by setting `hide_ids=YES` in the config, which turns all the Ids into `ftp` in the output. Hiding information about the system.
```ftp
150 Here comes the directory listing.
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```
In the olden days folks could brute force the usernames enumerated by FTP, but now that [fail2ban](https://en.wikipedia.org/wiki/Fail2ban) rate limiting is implemented its less of a problem.

# Downloading All available files 

We can use wget to download all the files on the system.
``` shell
becker63@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/                                         
           => ‘10.129.14.136/.listing’                                                                     
Connecting to 10.129.14.136:21... connected.                                                               
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.                                                                 
12.12.1.136/.listing           [ <=>                                  ]     466  --.-KB/s    in 0s       
```
# Uploading files 

Sometimes ftp will allow you to upload files, especially in web servers where developers push out code. This can lead to an `RCE` if not configured correctly (as it sometimes is, esp on web servers) so make sure you can upload to the server with the `put` command.

# Connecting to the server with telnet/[[Netcat]]
```shell
becker63@htb[/htb]$ nc -nv 10.129.14.136 21
```
```shell
becker63@htb[/htb]$ telnet 10.129.14.136 21
```

# Connecting to the server with openssl
```shell
becker63@htb[/htb]$ openssl s_client -connect 10.129.14.136:21 -starttls ftp
```
