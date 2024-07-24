---
description: 07/24/2024
---

# 🕸️ Web Hacking Reloaded

## It's Time to get Good at Web

Hello and welcome to my web series featuring guided exploitation tactics, techniques, and procedures. There is another web series page (the OG one), where I showcase many similar things. However, everything here will be updated and brought into a new light. I hope you enjoy!

If interested, you can check out the OG page below!

{% content-ref url="oscp/web-page-methodology/" %}
[web-page-methodology](oscp/web-page-methodology/)
{% endcontent-ref %}

## Jumping Into the Methodology

When it comes to testing and evaluating the security of a web application, there are a couple things to consider and keep in mind.&#x20;

{% embed url="https://appsec.guide/docs/web/burp/" %}
Burp Intro
{% endembed %}

### Step 1: Start off by Asking Yourself the Following

1. What is this app?
2. What does it do?
3. What is its purpose?
4. What can I do as an unauthenticated user vs am authenticated user?

### Step 2: Utilize Burp Suite and Map the Application

Begin to make requests using the various features and functionalities that the web app has to offer.&#x20;

Where does each request take you? How do different objects communicate with the underlying API? Is each object or request reaching out to the API in a different manner?

### Step 3: Utilize the Highlighting Feature in Burp

Large or complex web apps will often require lots of requests and mapping in order to understand the application and how it works. With that said, you may observe certain actions or requests that result in something interesting. Rather than having to remember the intricacies and specifics of that certain request, highlight it and that way you can color code it and find it immediately out of the bunch.

**To Highlight:**

* **Proxy** tab -> HTTP history -> Right-click request -> Select <mark style="color:yellow;">Highlight</mark>
* Also, be sure to rename requests within the **Repeater** tab

These highlights will be important later. Be sure to color-coordinate everything that is somewhat related (e.g. map out the "login flow" and use the same color for all of those).

### Step 4: Finding Secrets Through Strenuous Request Analysis

Look for Base64 encoding/decoding, look for user enumeration, PII, password hashing, etc.

Be sure to color code these as well using the highlighting methodology above.&#x20;

### Step 5: Authentication Techniques (e.g. JSON Web Token (JWT) Token)

This is generally a response technique for authenticating users. They are usually stored and encoded via Base64. We can then take this token and send it to the **Decoder** section in Burp. From there, we can decode as Base64 and instead of Hex, we want to use Text.

&#x20;
