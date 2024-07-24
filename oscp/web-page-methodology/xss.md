---
description: Cross-Site Scripting
---

# XSS

## Introduction

XSS is among some of the most common vulnerabilities throughout the Internet.

This is mainly due to the increased amount of user interaction in today's web applications.

How do they work?

* XSS take advantage of the fact that web apps execute scripts on users' browsers

XSS comes in three different forms

* <mark style="color:yellow;">Stored XSS</mark> (The code is stored on a database prior to execution)
* <mark style="color:yellow;">Reflected</mark> (The code is not stored in a database, but is reflected on the server)
* <mark style="color:yellow;">Document-Object Model (DOM)-Based</mark> (The code is both stored in a database and executed in the browser)

### Summary of XSS attacks

* Run a script in the browser that was not written by the web application owner
* Can run behind the scenes, without any visibility or require the user input to start execution
* Can obtain any type of data present in the current web application
* Occur as a result of improperly sanitized input being embedded in the UI
* Can be used to steal session tokens, leading to account takeover
* Can be used to draw DOM objects over the current UI, leading to the perfect phishing attack that cannot be identified by a non-technical user.

## XSS Discovery

Input field i.e. comment section?

```
<strong>Please improve your web application</strong>
```

* If this field is vulnerable, the text will be bolded!

Also try to create a pop up:

```
<script>alert(1)</script>
```

### Exploit Examples

See an input field?

```
<script>alert('XSS');</script>
```

* If you get a pop up window that says "XSS", this field is indeed vulnerable to an XSS attack

URL-based Injection

```
http://testsite.com/search?q=puppies<script>alert<('XSS')
```

## Reflected XSS Overview

* Look for anything and everything that has an input field
* Place the payload in there and see if you can get reflective behavior
* Attempt this kind of behavior anywhere

## Stored XSS Overview



* Look for anything and everything that has an input field
* Place the payload in there and see if you can get reflective behavior
* Attempt this kind of behavior anywhere
* Remember, this behavior is stored on the server which means it will impact other users
