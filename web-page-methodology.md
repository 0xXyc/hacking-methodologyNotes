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

## Login Page

Username:

```
' or 1=1-- -
```

Password:

```
' or 1=1-- -
```

## Web Application Attacks

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

Reviewing URLs:

* File extensions
* Routes
* Hidden web directories (robots.txt or sitemap.xml
* Non-standerd ports

Reviewing the content of the web page:

*
