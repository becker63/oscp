---
tags:
  - HTB-Academy
---
With the XML output, we can easily create HTML reports that are easy to read, even for non-technical people. This is later very useful for documentation, as it presents our results in a detailed and clear way. To convert the stored results from XML format to HTML, we can use the tool `xsltproc`.

```shell
xsltproc <nmap-output.xml> -o <nmap-output.html>
```
![[Pasted image 20240224123546.png]]