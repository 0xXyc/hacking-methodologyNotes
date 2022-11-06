# XML External Entities (XXE)

## What is XXE?

1. <mark style="color:yellow;">Attacking systems that parse XML input</mark>
2. Abuse SYSTEM entity and get malicious
3. <mark style="color:yellow;">Attacks include DoS, LFI, RCE, and more</mark>

Can you place malicious XML files onto a server and get the web application to parse the XML and trigger the attack?

### Test XML File for XXE Testing

Classic XXE:

{% code overflow="wrap" %}
```
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><root>&test;</root>
```
{% endcode %}

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE gift [
    <!ENTITY from "hacker">
]>

<gift>
    <To>Frank</To>
    <From>&from;</From>
    <Item>Baseball Cards</Items>

</gift>
```

### XXE Payloads (PayloadsAllTheThings)

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#billion-laugh-attack" %}

## XXE Attack and Defense

* You should attempt to identify file upload possibilities
* Once you identify a file upload, upload it to the web app and intercept the request with Burp
* If the XML is parsed, it will trigger the malicious code and return the results blindly (hence why you need Burp)

Response intercepted from Burp:

* you can see that we see the results of /etc/passwd

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

## Remediation

* If you disable external entities
* Once you do this, you will be able to mitigate this vulnerability
