---
description: The quieter you are, the more you can hear...
---

# Proxies/Man in The Middle (MiTM)

## MITM Proxy

### Installation

Apt:

`apt install mitmproxy`

Brew:

`brew install mitmproxy`

### Usage

FoxyProxy Configuration:

* For convenience, I chose to use FoxyProxy. However, if it is not working, we can go into the Proxy settings and add a Proxy configuration within settings:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>Settings -> Network (Mac)</p></figcaption></figure>

* Add a new proxy configuration for localhost running on port 8080
* Be sure to add HTTP/HTTPS

Web (GUI):

`mitmweb`

CLI:

`mitmproxy`

Curl:

`curl --proxy http://localhost:8080 https://example.com`

### Adding mitmproxy as a Certificate Authority (CA)

* To begin capturing HTTPS traffic, we need to add mitmproxy as a CA

Linux:

```
 wget -e https_proxy=127.0.0.1:8080 --ca-certificate ~/.mitmproxy/mitmproxy-ca-cert.pem https://example.com/
```

Mac:

```
sudo security add-trusted-cert -d -p ssl -p basic -k /Library/Keychains/System.keychain ~/.mitmproxy/mitmproxy-ca-cert.pem
```

If you are still stuck, use this guide. It is excellent:

{% embed url="https://earthly.dev/blog/mitmproxy/" %}

## Capabilities

### <mark style="color:yellow;">Certificate Pinning Bypass</mark>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>Certificate Pinning Bypasses</p></figcaption></figure>

#### <mark style="color:yellow;">What is Certificate Pinning and why is it bad for us?</mark>

> "Pinning is the process of associating a host with their expected X509 certificate or public key. Once a certificate or public key is known or seen for a host, the certificate or public key is associated or 'pinned' to the host. If one or more certificates or public keys are acceptable, then the program will hold a 'pinset'."

<mark style="color:yellow;">**When do you pin?**</mark>

> "You should pin anytime you want to be relatively certain of the remote host's identity or when operating in a hostile environment. Since one or both are almost always true, you should probably pin all the time."

{% embed url="https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning" %}

### References

{% embed url="https://quickref.me/mitmproxy" %}

