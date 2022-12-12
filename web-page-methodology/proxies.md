# Proxies

## MITM Proxy

### Installation

Apt:

`apt install mitmproxy`

Brew:

`brew install mitmproxy`

### Usage

FoxyProxy Configuration:

* For convenience, I chose to use FoxyProxy. However, if it is not working, we can go into the Proxy settings and add a Proxy configuration within settings:

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>Settings -> Network (Mac)</p></figcaption></figure>

* Add a new proxy configuration for localhost running on port 8080
* Be sure to add HTTP/HTTPS

Web (GUI):

`mitmweb`

CLI:

`mitmproxy`

Curl:

`curl --proxy http://localhost:8080 https://example.com`

### Capabilities



### References

{% embed url="https://quickref.me/mitmproxy" %}

