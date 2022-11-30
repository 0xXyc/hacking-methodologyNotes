# "Sorcerer" (Intermediate)

<figure><img src="../../../.gitbook/assets/image (4) (4).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 812a4224b590a1ce9bace74e1d6db4c6 (RSA)
|   256 d0732a05527f89093776e356c8ab2099 (ECDSA)
|_  256 3a2dde33b01ef2350f8dc8d78ff9e00e (ED25519)
80/tcp    open  http     nginx
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100003  3           2049/udp   nfs
|   100003  3,4         2049/tcp   nfs
|   100005  1,2,3      45519/tcp   mountd
|   100005  1,2,3      50443/udp   mountd
|   100021  1,3,4      43329/tcp   nlockmgr
|   100021  1,3,4      50121/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/udp   nfs_acl
2049/tcp  open  nfs_acl  3 (RPC #100227)
7742/tcp  open  http     nginx
|_http-title: SORCERER
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
8080/tcp  open  http     Apache Tomcat 7.0.4
|_http-title: Apache Tomcat/7.0.4
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Apache Tomcat
43329/tcp open  nlockmgr 1-4 (RPC #100021)
45519/tcp open  mountd   1-3 (RPC #100005)
54375/tcp open  mountd   1-3 (RPC #100005)
55683/tcp open  mountd   1-3 (RPC #100005)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

* Vhost scan
* Directory bruteforce
* Source code analysis

## Enumeration

### Port 80 - HTTP (nginx)

Nothing here at the moment

### Port 8080 - HTTP (Tomcat 7.0.4)

* Searchsploit

```
searchsploit tomcat 7.0
```

XSS exploit for 7.0.4

### Port 7742 - HTTP (Nginx) - Sorcerer

<figure><img src="../../../.gitbook/assets/image (5) (9).png" alt=""><figcaption></figcaption></figure>

Directory Bruteforce:

```
gobuster dir -u http://192.168.81.100:7742 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.81.100:7742
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2022/11/13 19:37:31 Starting gobuster in directory enumeration mode
===============================================================
/default              (Status: 301) [Size: 178] [--> http://192.168.81.100:7742/default/]
/index.html           (Status: 200) [Size: 1219]
/zipfiles             (Status: 301) [Size: 178] [--> http://192.168.81.100:7742/zipfiles/]
Progress: 4694 / 4714 (99.58%)===============================================================
2022/11/13 19:37:52 Finished
```

* /zipfiles

<figure><img src="../../../.gitbook/assets/image (6) (4).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">/home/max/.ssh/</mark>
* In max's directory, I found SSH keys and a program named scp\_wrapper.sh
* I grabbed the SSH key and attempted to SSH in as max with the private key with no luck

<figure><img src="../../../.gitbook/assets/image (1) (6).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Taking Advantage of SCP and Overwriting Existing Script on Target

* By modifying the script locally (on Kali), we can SCP it back to the target machine with the same name and have the script execute the "bash" binary rather than the "scp" one.
* This can simply be done by modifying "scp" in the script and replacing it with "bash"

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
