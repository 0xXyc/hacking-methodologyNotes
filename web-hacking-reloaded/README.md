---
description: 07/24/2024
---

# 🕸️ Web Hacking Reloaded

## It's Time to get Good at Web

Hello and welcome to my web series featuring guided exploitation tactics, techniques, and procedures. There is another web series page (the OG one), where I showcase many similar things. However, everything here will be updated and brought into a new light. I hope you enjoy!

If interested, you can check out the OG page below!

{% content-ref url="../oscp/web-page-methodology/" %}
[web-page-methodology](../oscp/web-page-methodology/)
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

<figure><img src="../.gitbook/assets/image (223).png" alt=""><figcaption><p>Example of a raw JWT Token before Decoding process</p></figcaption></figure>

#### Once Decoded...

Ensure that there isn't any PII that is being leaked within the JWT token.&#x20;

If there is, this is a finding, it is a vulnerability, and needs to be reported.

Also, verify how long the JWT tokens last and are valid. If it is a forever-lasting JWT token, this is a problem.&#x20;

## Exploiting Indirect Object Reference (IDOR)

<figure><img src="../.gitbook/assets/image (224).png" alt=""><figcaption><p>This is an object reference</p></figcaption></figure>

This object reference becomes a problem when it is not sanitized correctly, allowing for an attacker to pass an arbitrary object reference (number) of their choice, in order to indirectly access other objects or pieces of information on the site.&#x20;

### How to Exploit

Simply put, when you see a number or an `id` at the end of a `GET` request, attempt IDOR and simply just change the number to see if you gain access to a different resource.&#x20;

The best way to exploit this is utilizing the **Repeater** tab and then you can manipulate the value and view the effect that it plays on the server in the **Response** tab.

### Automating This Attack w/ the Intruder tab

Within the screenshot below, you can see that we are within the **Intruder** tab, we modify the `id` value that we identified to be an IDOR value where we can add a payload value and automate an attack type, _Sniper_, in order to start this attack and dynamically fuzz this field through a list of values.&#x20;

<figure><img src="../.gitbook/assets/image (225).png" alt=""><figcaption><p>Showcasing Sniper Attack Type within the Intruder tab</p></figcaption></figure>

### Specifying Payload Set

We can set a range of numbers within the **Payload type** that will allow you to increment or decrement as needed.

<figure><img src="../.gitbook/assets/image (226).png" alt=""><figcaption><p>Payload Configuration</p></figcaption></figure>

Also, be sure to uncheck the **Payload Encoding** feature

<figure><img src="../.gitbook/assets/image (227).png" alt=""><figcaption><p>This can ruin your attack and make your requests invalid</p></figcaption></figure>

### Leveraging Advanced Intruder Configuration

**If this section isn't clear or gets a little confusing, be sure to check out minute 28 on this video:**

{% embed url="https://www.youtube.com/watch?v=QiNLNDSLuJY" %}

Instead of looking at the results within the **Results** tab manually, we can go into the **Settings** tab and then scroll down to the **Grep - Extract** section. Select the "Extract the following items from responses" checkbox and select "Add".

Upon selecting it, we can select **Fetch/Refetch Response**.&#x20;

From here, we can essentially utilize `grep` to parse out the important information that we want.

For example, in the provided screenshot, we will carve out the data that begins after `UserId` and stops at `createdAt`.&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Showcasing the <code>grep</code> feature for parsing out specific, important data from a request</p></figcaption></figure>

You can then click **Ok** to append the `grep` and parse out the requests.

To view this parsed data, go to the **Results** tab. This will then show our **Start after expression** section that we specified above and allow us to view it within the **Results** tab.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Showcasing our parsed data, <code>UserId</code> above</p></figcaption></figure>

#### Intruder Thoughts

Intruder is extremely important when it comes to extracting important, specific, data from your requests.

### `grep`'ing for multiple fields to extract important data

You can also add multiple different fields by adding them and specifying them within the **Grep - Extract** setting.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Showcasing the usage of multiple fields</p></figcaption></figure>

**Once finished, you can start the attack by the following:**

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Showcasing the attack!</p></figcaption></figure>

**Once we kick off our attack, we can now see our specified fields:**

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Showcasing our fields specified above in the <code>grep</code> section</p></figcaption></figure>

## Finding Logical Flaws Within Websites

### What is a Logic Flaw?

A logic flaw occurs when a developer for a webapp or a site automatically assumes that all users are going to follow the same procedures, steps, and methods in order to accomplish something. Essentially, assuming all users will be utilizing the same workflow as each other.&#x20;

**The Goal?**

We want to see if we can achieve something before you are actually supposed to.

To-Do:

I need to more in depth into this.
