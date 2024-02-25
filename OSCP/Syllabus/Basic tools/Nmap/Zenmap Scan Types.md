## <mark class="hltr-green">Intense Scan</mark>
```shell
nmap -T4 -A -v <target>
```
## <mark class="hltr-pink">Intense scan plus UDP</mark>
``` shell
nmap -sS -sU -T4 -A -v <target>
```
## <mark class="hltr-red">Intense scan, all TCP ports</mark>
```shell
nmap -p 1-65535 -T4 -A -v <target>
```
## <mark class="hltr-orange">Intense scan, no ping </mark>
```shell
nmap -T4 -A -v -Pn <target>
```
## <mark class="hltr-cyan">Ping scan</mark>
```shell
nmap -sn <target>
```
## <mark class="hltr-blue">Quick scan</mark>
```shell
nmap -T4 -F <target>
```
## <mark class="hltr-green">Quick scan plus</mark>
``` shell
nmap -sV -T4 -O -F --version-light <target>
```
## <mark class="hltr-pink">Quick traceroute</mark>
```shell
nmap -sn --traceroute <target>
```
## <mark class="hltr-orange">Slow comprehensive scan</mark>
```shell
nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389 -PU40125 -PY -g 53 --script "default or (discovery and safe)" <target>
```
