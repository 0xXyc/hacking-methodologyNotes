# Directory Bruteforcing

## Dirsearch

```
dirsearch -u http://<IP>
```

## Gobuster

* I will organize this by the utilization of different wordlists

<mark style="color:yellow;">common.txt</mark>:

```
locate common.txt

gobuster dir -u http://"IP"/ -w /usr/share/wordlists/dirb/common.txt --no-error -x asp,php
```

<mark style="color:yellow;">gobustergob-raft-small-words</mark>:

```
gobuster dir -u http://"IP"/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt --no-error -x asp,php
```

directory-list-2.3-small.txt:

```
gobuster dir -u http://"IP"/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --no-error -x asp,php
```

## Wfuzz

<mark style="color:yellow;">raft-large-files.txt</mark>

```
wfuzz -c -z file,/opt/SecLists/Discovery/Web-Content/raft-large-files.txt --hc 404 "$URL" 
```
