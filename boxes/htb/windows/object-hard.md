---
description: 10-21-2022
---

# Object (Hard)

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (2) (3).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:red;">We are able to create an account!!!</mark>
* <mark style="color:yellow;">/robots.txt</mark>, let's go investigate the contents
  * Nothing of interest
* I attempted to access the Groovy script section of Jenkins but I do not have the correct permissions to access
  * This can be attempted by going to <mark style="color:yellow;">/script</mark>

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

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

### Evil-WinRM FTW

```
evil-winrm -i object.htb -u oliver
Password: 
```

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
