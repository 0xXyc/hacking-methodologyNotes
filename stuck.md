---
description: What has worked in the past that might work now?
---

# 🤕 Stuck?

## SSH Open?

* Can you port forward anything and expose it to Kali?
* Did you enumerate all possible users?
  * username:password(same as username)&#x20;

```
ssh eleanor@192.168.81.100
Password: eleanor
```

## Pathways

* Realize that sometimes you need to go off of the most traveled path to realize that there is another or better way to exploit something
* <mark style="color:yellow;">Remember</mark>: it may look overly complex on purpose but what if the developer made a silly mistake somewhere?

## Searchsploit

* Take note of every service, application, or program
* Searchsploit all of them with and without the version numbers (if applicable)

## Passwords

* Default passwords?
* Google the service for default passwords
* Perform static analysis on web app source code for mistakes, hard-coded credentials, etc.
* Weak passwords?
* Authentication bypass?
* Sensitive information disclosure?

## Web Servers

Directory Bruteforcing:

* Run the endpoints through more than one wordlist
*
