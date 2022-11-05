---
description: Love/Hate relationship with web pages
---

# 🕸 Web Page Methodology

## Checklist

1. <mark style="color:yellow;">Visual Inspection/"mess around" with web application</mark> -- Take note of what the normal/expected behavior is. What seems off or broken? What is the web server's native programming language? Check robots.txt.
2. <mark style="color:yellow;">Dirsearch</mark> -- Wordlist variety (default and medium wordlist), specify proxy (if needed), use correct port and path
3. <mark style="color:yellow;">Nikto</mark> -- Vulnerability scan/easy win?
4. <mark style="color:yellow;">Wappalyzer</mark> -- What versions are running? <mark style="color:yellow;">Google</mark> and <mark style="color:yellow;">Searchsploit</mark> them all.
5. <mark style="color:yellow;">Source Code</mark> -- Is there anything within the source code? Easy wins via usernames, emails, domains, secret comments?

## OWASP Top 10

1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)

## Login Page

Username:

```
' or 1=1-- -
```

Password:

```
' or 1=1-- -
```

## "The Journey to Try Harder" - Web Application Attacks

As a pentester, you need to gather information about the web application.

While testing a web app, you should be <mark style="color:yellow;">constantly asking yourself these questions</mark>:

* What is the purpose of the application?
* What language is the web application written in?
* What version is the web application running?
* How is the web application being hosted?
* Does the web application connect to a db? If yes, what is the software that the db is using and what is the version?

Once you have identified the components of the web application, this will allow you to proceed to the next phase by enumerating the components/issues you identified instead of blindly running an exploit against the web app.

* Enumeration is crucial for reviewing all possible attack vectors that could compromise the web application

### Things to check for when enumerating a web app:

<mark style="color:yellow;">Reviewing URLs</mark>:

* File extensions
* Routes
* Hidden web directories (robots.txt or sitemap.xml
* Non-standard ports

<mark style="color:yellow;">Reviewing the content of the web page</mark>:

* Always review the source code of the web page
* Inspect every element to see how the web app works
* Review the request and response headers to understand how the web application behaves wehn you make certain actions to it
* Check for admin consoles (Wordpress applications will have a directory /admin that can be used to access the Wordpress admin console)

### BurpSuite Trainings

{% embed url="https://www.youtube.com/playlist?list=PLqG-wtrX3aA_wYTrnDHoCBkKBoI4z9oLd" %}
Secure Ideas
{% endembed %}

{% embed url="https://www.bugcrowd.com/resource/introduction-to-burp-suite/" %}
Jason Haddix Webinar
{% endembed %}

### Nikto Usage

A web server scanner that performs tests against the web server for multiple items. This tool is not only used to scan for vulnerabilities on the web application, but checks for server configuration that includes multiple index files, HTTP server options, and will attempt to identify the version installed.

* <mark style="color:yellow;">Very noisy</mark>

### HTTPIe

A tool that is designed for testing, debugging, and interacting with API's and HTTP servers. The HTTP and HTTPs commands allow for creating and sending arbitrary HTTP requests.

## Exploits

### Exploiting Admin Consoles

When an administrative login panel is left exposed, it can make it significantly easier for hackers to compromise that site. However, this is dependent on the security configurations and permissions that the developer/application have implemented.&#x20;

As pentesters, we can execute techniques such as:

* Bruteforcing
* Signing in with compromised credentials/obtained credentials
* Exploiting an unpatched and vulnerable system

{% embed url="https://www.exploit-db.com/search?q=Authentication+Bypass" %}
Examples of authentication bypass
{% endembed %}

### Exploit Examples

* CASAP Automated Enrollment System: [https://www.exploit-db.com/exploits/49463](https://www.exploit-db.com/exploits/49463)
* Alumni Management System 1.0 [https://www.exploit-db.com/exploits/48883](https://www.exploit-db.com/exploits/48883)
* Online Hotel Reservation System 1.0 [https://www.exploit-db.com/exploits/49420](https://www.exploit-db.com/exploits/49420)

Vulnerability-Specific:

* <mark style="color:yellow;">OWASP cross-site scripting (XSS)</mark>: [https://www.owasp.org/index.php/Cross-site\_Scripting\_(XSS)](https://www.owasp.org/index.php/Cross-site\_Scripting\_\(XSS\))
* <mark style="color:yellow;">OWASP Directory Traversal Vulnerabilities</mark>: [https://owasp.org/www-community/attacks/Path\_Traversal](https://owasp.org/www-community/attacks/Path\_Traversal)
* <mark style="color:yellow;">SQL Injections: OWASP</mark>: [https://www.owasp.org/index.php/SQL\_Injection](https://www.owasp.org/index.php/SQL\_Injection)
* Pentest Monkey SQL Cheat Sheets: [http://pentestmonkey.net/category/cheat-sheet/sql-injection](http://pentestmonkey.net/category/cheat-sheet/sql-injection)
* <mark style="color:yellow;">File Inclusion Vulnerabilities (Metaploit Unleashed)</mark>: [https://www.offensive-security.com/metasploit-unleashed/file-inclusion-vulnerabilities/](https://www.offensive-security.com/metasploit-unleashed/file-inclusion-vulnerabilities/)&#x20;
* <mark style="color:yellow;">OSWAP Testing for LFI</mark>: [https://owasp.org/www-project-web-security-testing-guide/latest/4-Web\_Application\_Security\_Testing/07-Input\_Validation\_Testing/11.1-Testing\_for\_Local\_File\_Inclusion](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web\_Application\_Security\_Testing/07-Input\_Validation\_Testing/11.1-Testing\_for\_Local\_File\_Inclusion)

Hands on areas to improve your web attack skills:

* Metasploitable 2

{% embed url="https://metasploit.help.rapid7.com/docs/metasploitable-2" %}

Guide:

[https://metasploit.help.rapid7.com/docs/metasploitable-2-exploitability-guide](https://metasploit.help.rapid7.com/docs/metasploitable-2-exploitability-guide)

Also do TCM's web security course again!

Overthewire:

[http://overthewire.org/wargames/natas/](http://overthewire.org/wargames/natas/)

\
