---
description: 10/01/2024
cover: >-
  https://theintercept.com/wp-content/uploads/2016/08/Pokemon_theintercept_01.gif?fit=1439%2C720
coverY: 0
---

# Hacking API's

## Hacking Application Programming Interfaces!!

Wow, just in time for the spooky season, let's dive right into this one!

Notably, I am going to be using some personal experience as well as a few references throughout the writing of this blog post. I hope you enjoy!!

## Introduction

In many assessments, your client or target is going to have some sort of way to process information between their web apps and in a standardized way where it is understood what to do with that data.

### Picture This... 👀💭

Imagine a bad guy steals a blueprint of one of your important buildings and assets.&#x20;

<mark style="color:yellow;">Similarly, if an attacker was able to find API documentation for one of your API's and web applications, they would slowly, but surely, start to begin to understand how your API works as well as how it can be exploited</mark>.&#x20;

## How can an API endpoint be Exploited Using API Documentation?

**First thing's first:** Always trigger Burp and activate the proxy.

Begin navigate the web app as a normal user to utilize the _**Repeater**_ and _**Proxy**_** functionalities** and begin to look for API endpoints. This can be commonly seen be captured within the `URL` body of the requests within Burp. For example, `/api/user/bob`. Also, the HTTP method may be something different than a traditional `GET`. It could be a `PATCH` or a `POST`.&#x20;

For example, if you do not get any at first, don't fret. Test out and enumerate additional functionality such as updating your email address or something else.&#x20;

### Fuzzing

So, once you pinpoint the API endpoint, (for this example, we're going to use `/api`), you're going to want to be able to fuzz each and every known or possible endpoint that could possibly be valid/exist.&#x20;

**(e.g.):**

```
/api/user/FUZZ
/api/FUZZ
```

Make it a mission to find or compile a wordlist that will help you discover and enumerate a documentation endpoint for an endpoint.&#x20;

You can manually fuzz or you can run automated fuzz tooling to enumerate the potential endpoints.

If you are unsure, utilize ChatGPT to create some possible endpoint names that can be passed to your fuzzer in the form of a wordlist.&#x20;

### Found API Documentation Endpoint?

At this point, you will need to reach out to the client/target and ask if they absolutely are required to provide API documentation to the public. This is totally fine.&#x20;

The only time problems arise are when authentication checks such as auth tokens and CSRF tokens are not provided within the requests, which mean that anyone can make unauthenticated API requests.

Keep in mind that you can also hide endpoints from normal users by restricting access to the API documentation by providing authentication or using some sort of meta tags so that these endpoints do not get followed by web crawlers.&#x20;

## Swagger API Documentation

**Coming soon.**

## References

{% embed url="https://www.youtube.com/watch?v=sl1yqGhuVy4" %}

{% embed url="https://blog.vidocsecurity.com/blog/hacking-swagger-ui-from-xss-to-account-takeovers/" %}

{% embed url="https://medium.com/@ghostlulzhacks/swagger-api-c07eca05441e" %}
Wow, this looks awesome.
{% endembed %}

{% embed url="https://www.youtube.com/watch?v=Ck4lCRpa1XQ" %}

{% embed url="https://www.youtube.com/watch?v=M_guA0wjrLg" %}
API FUZZING
{% endembed %}

{% embed url="https://www.youtube.com/watch?v=AxzpOVS23o8" %}

{% embed url="https://www.youtube.com/watch?v=k5HZI6CfHw4" %}

{% embed url="https://www.youtube.com/watch?v=85vdKS0vNN0" %}

{% embed url="https://www.youtube.com/watch?v=3Z2STZGqvc4" %}
