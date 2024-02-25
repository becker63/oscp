---
tags:
  - HTB-Academy
  - web-guide
---
# What is SMB?
* SMB is a file transfer protocol mostly used for a the windows operating system. 
* Uses TCP 
* very old
	* Every version of windows can communicate through this protocol 
* Not necessarily based off the systems file structure 
	* Provides access to arbitrary parts of the file system 
* uses ports `137`, `138`, `139` and `445`
* ## ACL 
	* Access to the server is controled by an ACL 
	* [Example of setting one up](https://docs.netapp.com/us-en/ontap/smb-admin/create-share-access-control-lists-task.html#:~:text=Configuring%20share%20permissions%20by%20creating,UNIX%20user%20or%20group%20names.)
	* `execute`, `read`, or `full access` for individual users or user groups.

# Whats a workgroup?
A workgroup is just a network name. 

*"Workgroup is the network name. On Windows, WORKGROUP is the default name Microsoft gives, but you can set it to anything. As long as all the computers are on the same workgroup, they can see each other. On Linux, when you look under Network in your file manager, you should be able to freely choose which workgroup you want to browse (it will be a folder and inside it are the computers in that workgroup)." -GoldenDreamcast From [this](https://www.reddit.com/r/linuxquestions/comments/ab0gss/setting_up_samba_what_is_the_workgroup_for/) reddit post *
# Samba 
* Linux implementation of SMB 
* based off the `Common Internet File System` ([CIFS](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs/934c2faa-54af-4526-ac74-6a24d126724e)) network protocol.
	* a "dialect" of SMB.
		* a very specific implementation of the SMB protocol
* usually is referred to as `SMB / CIFS`
* When talking to older netbios based services (<2000s windows machinves) will use ports `137`, `138`, `139`
	* when talking to newer machines will only use port `445`
* ## Active directory 
	* Samba 3 can be a member of a AD domain 
	* Samba 4 can be an AD domain controller

## Samba settings 

|**Setting**|**Description**|
|---|---|
|`[sharename]`|The name of the network share.|
|`workgroup = WORKGROUP/DOMAIN`|Workgroup that will appear when clients query.|
|`path = /path/here/`|The directory to which user is to be given access.|
|`server string = STRING`|The string that will show up when a connection is initiated.|
|`unix password sync = yes`|Synchronize the UNIX password with the SMB password?|
|`usershare allow guests = yes`|Allow non-authenticated users to access defined share?|
|`map to guest = bad user`|What to do when a user login request doesn't match a valid UNIX user?|
|`browseable = yes`|Should this share be shown in the list of available shares?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`read only = yes`|Allow users to read files only?|
|`create mask = 0700`|What permissions need to be set for newly created files?|

```shell
becker63@htb[/htb]$ cat /etc/samba/smb.conf | grep -v "#\|\;" 

[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

## Dangerous settings 

|**Setting**|**Description**|
|---|---|
|`browseable = yes`|Allow listing available shares in the current share?|
|`read only = no`|Forbid the creation and modification of files?|
|`writable = yes`|Allow users to create and modify files?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`enable privileges = yes`|Honor privileges assigned to specific SID?|
|`create mask = 0777`|What permissions must be assigned to the newly created files?|
|`directory mask = 0777`|What permissions must be assigned to the newly created directories?|
|`logon script = script.sh`|What script needs to be executed on the user's login?|
|`magic script = script.sh`|Which script should be executed when the script gets closed?|
|`magic output = script.out`|Where the output of the magic script needs to be stored?|
# Connecting to samba 
Lets take a look at how we would connect to and use the service
## Anonymous (Null) samba connection dir listing

*"Now we can display a list (`-L`) of the server's shares with the `smbclient` command from our host. We use the so-called `null session` (`-N`), which is `anonymous`  access without the input of existing users or valid passwords." -HTB*
```shell
becker63@htb[/htb]$ smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```
## Connecting to the samba share 
```shell
becker63@htb[/htb]$ smbclient //10.129.14.128/notes

Enter WORKGROUP\<username>'s password: 
Anonymous login successful
Try "help" to get a list of possible commands.


smb: \>
```
## Downloading and viewing a file from a samba share
```shell
smb: \> get prep-prod.txt 

getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec) 
(average 8,7 KiloBytes/sec)


smb: \> !ls

prep-prod.txt


smb: \> !cat prep-prod.txt

[] check your code with the templates
[] run code-assessment.py
[] …	
```
