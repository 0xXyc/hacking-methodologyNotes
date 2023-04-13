# Squashed (Easy)

<figure><img src="../../../.gitbook/assets/Squashed.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Built Better
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      36214/udp6  mountd
|   100005  1,2,3      37093/tcp6  mountd
|   100005  1,2,3      42899/tcp   mountd
|   100005  1,2,3      48117/udp   mountd
|   100021  1,3,4      39287/tcp6  nlockmgr
|   100021  1,3,4      43699/tcp   nlockmgr
|   100021  1,3,4      46570/udp   nlockmgr
|   100021  1,3,4      53611/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

* <mark style="color:yellow;">SSH</mark> is a possible entry point
* <mark style="color:yellow;">HTTP</mark> is a possible entry point (Apache 2.4.41)
* 2049 <mark style="color:yellow;">NFS</mark> -- Must enumerate for information disclosure etc.

## Enumeration

### Port 80 - HTTP (Apache 2.4.41)

* Added IP address of machine to /etc/hosts under squashed.htb

Directory Bruteforce (Dirsearch):

```
dirsearch -u http://squashed.htb
```

* No lucrative findings

### Port 2049 - NFS



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

##

