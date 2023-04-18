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

#### Dirsearch:&#x20;

```
dirsearch -u photobomb.htb
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
