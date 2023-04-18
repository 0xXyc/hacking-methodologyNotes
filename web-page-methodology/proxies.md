---
description: The quieter you are, the more you can hear...
cover: https://lucaslegname.github.io/assets/images/mitmproxy/how-mitmproxy-works.jpg
coverY: 0
---

# Proxies/Man in The Middle (MiTM)

## MITM Proxy (Targetting HTTPS/SSL)

### Installation

Apt:

`apt install mitmproxy`

Brew:

`brew install mitmproxy`

### Usage

FoxyProxy Configuration:

* For convenience, I chose to use FoxyProxy. However, if it is not working, we can go into the Proxy settings and add a Proxy configuration within settings:

<figure><img src="../.gitbook/assets/image (1) (13).png" alt=""><figcaption><p>Settings -> Network (Mac)</p></figcaption></figure>

* Add a new proxy configuration for localhost running on port 8080
* Be sure to add HTTP/HTTPS

<mark style="color:yellow;">Web (GUI):</mark>

`mitmweb`

<mark style="color:yellow;">CLI:</mark>

`mitmproxy`

<mark style="color:yellow;">Dump:</mark>

`mitmdump -w outfile`

<mark style="color:yellow;">Curl:</mark>

`curl --proxy http://localhost:8080 https://example.com`

<mark style="color:yellow;">Client Replay Attack:</mark>

* I think this is a great option for producing PoC's and sending them directly to the customer for proof and remediation

`mitmdump -nC outfile`

<figure><img src="../.gitbook/assets/image (2) (1) (2).png" alt=""><figcaption><p>General Usages</p></figcaption></figure>

### Adding mitmproxy as a Certificate Authority (CA)

* To begin capturing HTTPS traffic, we need to add mitmproxy as a CA

Linux:

```
 wget -e https_proxy=127.0.0.1:8080 --ca-certificate ~/.mitmproxy/mitmproxy-ca-cert.pem https://example.com/
```

* Then add this file to Firefox's Certificates
* Settings -> Search "Certificates" -> View Certificates -> Import

Mac (Very easy and automatic):

```
sudo security add-trusted-cert -d -p ssl -p basic -k /Library/Keychains/System.keychain ~/.mitmproxy/mitmproxy-ca-cert.pem
```

If you are still stuck, use this guide. It is excellent:

{% embed url="https://earthly.dev/blog/mitmproxy/" %}

## Capabilities

### <mark style="color:yellow;">Intercepting Requests</mark>

* Simply put, an intercepted request is paused so that a user can modify or discard the request before sending it the the target server
* `set intercept` <mark style="color:orange;">will configure interceptions (i) by default</mark>
* `~u /whatyouarelookingfor & ~q`
* Now, curl with the proxy flag and you will intercept the request in flight before it is sent to the server

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption><p>Curl Command</p></figcaption></figure>

* Click on the Red (intercepted) GET request

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption><p>Intercepted GET</p></figcaption></figure>

* To forward the request and complete the connection, press a to resume the flow
* Upon returning back to the Terminal window where you executed the curl command, you will see that you will get a request containing the weather!

<figure><img src="../.gitbook/assets/image (1) (2) (4).png" alt=""><figcaption><p>Flow continued and you get information back from the server</p></figcaption></figure>

### <mark style="color:yellow;">Certificate Pinning Bypass</mark>

<figure><img src="../.gitbook/assets/image (4) (14).png" alt=""><figcaption><p>Certificate Pinning Bypasses</p></figcaption></figure>

#### <mark style="color:yellow;">What is Certificate Pinning and why is it bad for us?</mark>

> "Pinning is the process of associating a host with their expected X509 certificate or public key. Once a certificate or public key is known or seen for a host, the certificate or public key is associated or 'pinned' to the host. If one or more certificates or public keys are acceptable, then the program will hold a 'pinset'."

<mark style="color:yellow;">**When do you pin?**</mark>

> "You should pin anytime you want to be relatively certain of the remote host's identity or when operating in a hostile environment. Since one or both are almost always true, you should probably pin all the time."

{% embed url="https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning" %}

### References

{% embed url="https://quickref.me/mitmproxy" %}
Reference sheet
{% endembed %}

{% embed url="https://docs.mitmproxy.org/stable/" %}
Main wiki for mitmproxy
{% endembed %}
