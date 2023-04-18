# Trick (Easy)

<figure><img src="../../../.gitbook/assets/Trick.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE    SERVICE        VERSION
22/tcp    open     ssh            OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 61ff293b36bd9dacfbde1f56884cae2d (RSA)
|   256 9ecdf2406196ea21a6ce2602af759a78 (ECDSA)
|_  256 7293f91158de34ad12b54b4a7364b970 (ED25519)
25/tcp    open     smtp?
|_smtp-commands: Couldn't establish connection on port 25
53/tcp    open     domain         ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp    open     http           nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
99/tcp    filtered metagram
100/tcp   filtered newacct
587/tcp   filtered submission
720/tcp   filtered unknown
765/tcp   filtered webster
992/tcp   filtered telnets
993/tcp   filtered imaps
1025/tcp  filtered NFS-or-IIS
1030/tcp  filtered iad1
1094/tcp  filtered rootd
1100/tcp  filtered mctp
1666/tcp  filtered netview-aix-6
2607/tcp  filtered connection
3367/tcp  filtered satvid-datalnk
4446/tcp  filtered n1-fwp
5101/tcp  filtered admdog
6106/tcp  filtered isdninfo
8652/tcp  filtered unknown
9595/tcp  filtered pds
25735/tcp filtered unknown
61532/tcp filtered unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
PORT      STATE         SERVICE
53/udp    open          domain
```

Notes:

* 22 is open, meaning, a possible entry point
* 25 SMTP is open -- Not tons of experience here, let's leave enumerating this for last
* 53 is open -- let's attempt zone transfer
* 80 is open -- let's run web enumeration in the background
  * Added trick.htb to /etc/host

## Enumeration

### Port 80 - HTTP (nginx 1.14.2)

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

#### ffuf Subdomain Enumeration

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.trick.htb" -u http://trick.htb -fs 5480

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://trick.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt
 :: Header           : Host: FUZZ.trick.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 5480
________________________________________________

:: Progress: [151265/151265] :: Job [1/1] :: 1129 req/sec :: Duration: [0:02:17] :: Errors: 0 ::
```

* No results

#### Dirsearch Directory Bruteforce

```
direarch -u http://trick.htb

Target: http://trick.htb/

[14:25:19] Starting: 
[14:25:19] 301 -  185B  - /js  ->  http://trick.htb/js/
[14:25:29] 301 -  185B  - /assets  ->  http://trick.htb/assets/
[14:25:29] 403 -  571B  - /assets/
[14:25:32] 301 -  185B  - /css  ->  http://trick.htb/css/
[14:25:35] 200 -    5KB - /index.html
[14:25:36] 403 -  571B  - /js/
```

* No useful information

#### Web Server Fingerprinting

{% code overflow="wrap" %}
```
whatweb trick.htb
http://trick.htb [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.14.2], IP[10.129.227.180], Script, Title[Coming Soon - Start Bootstrap Theme], nginx[1.14.2]

```
{% endcode %}

* No useful information

### Port 53 - DNS

#### DNS Zone Transfer

<pre><code>dig axfr trick.htb @10.129.227.180

; &#x3C;&#x3C;>> DiG 9.18.12-1-Debian &#x3C;&#x3C;>> axfr trick.htb @10.129.227.180
;; global options: +cmd
trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.		604800	IN	NS	trick.htb.
trick.htb.		604800	IN	A	127.0.0.1
trick.htb.		604800	IN	AAAA	::1
<strong>preprod-payroll.trick.htb. 604800 IN	CNAME	trick.htb.
</strong>trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 36 msec
;; SERVER: 10.129.227.180#53(10.129.227.180) (TCP)
;; WHEN: Tue Apr 18 14:34:22 EDT 2023
;; XFR size: 6 records (messages 1, bytes 231)
</code></pre>

* <mark style="color:yellow;">We find 3 confirmed subdomains</mark>

```
trick.htb
preprod-payroll.trick.htb
root.trick.htb
```

* Be sure to add these to /etc/hosts
* We now need to conduct enumeration on these new endpoints since our attack surface has increased

### Additional Port 80 Enumeration

#### preprod-payroll.trick.htb:

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs

