---
description: File Transfer Protocol (FTP)
---

# 🗃 21/FTP Enumeration

## Checklist

* [ ] Anonymous access?
* [ ] File read/file write?
* [ ] Can we READ any sensitive files that contain hints of creds?
* [ ] Can we WRITE a file and access it via browser? If so can we upload a webshell and gain access?
  * File upload → Execute from Web → Webshell
* [ ] Attempt password checking if you found credentials from other enumeration. If so, this will allow you to be able to gain access to a higher privileged session that can then allow write access if not obtained before.

### Banner Grabbing:

```python
nc -vn <IP> 21
```

### Anonymous Login:

```python
ftp <IP>

> anonymous
> anonymous
> ls -a # List all files (even hidden) (yes, they could be hidden)
> binary #Set transmission to binary instead of ascii
> ascii #Set transmission to ascii instead of binary
> bye #exit
```

### Nmap:

```python
nmap --script ftp-* -p 21 <IP>

#Note: This will still be done with the -sC option in nmap.
```

### Attempt Connection via Browser — Enter the following into the URL:

```python
<ftp://anonymous>:anonymous@<IP>
```

* Note: that if a web application is sending data controlled by a user directly to a FTP server you can send double URL encode %0d%0a (in double URL encode this is %250d%250a) bytes and make the FTP server perform arbitrary actions. One of this possible arbitrary actions is to download content from a users controlled server, perform port scanning or try to talk to other plain-text based services (like http)

### Downloading all files from FTP:

```python
wget -m <ftp://anonymous>:anonymous@<IP> #Download all
wget -m --no-passive <ftp://anonymous>:anonymous@<IP> #Download all
```

## Tools

nmap

wget

netcat

ftp client
