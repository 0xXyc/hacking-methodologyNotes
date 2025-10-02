---
description: 09/01/2025
---

# ✅ External Reconnaissance

## Importance of Recon

This is a vital phase of the offensive operation that allows you to gain intel before performing intrusive behaviors and other TTP's (Tactics, Techniques, and Procedures).

This reconnaissance phase is so important because it _**provides information**_ that will be leveraged to exploit the target or gain access to data.

### Organizational and Technical Reconnaissance

#### Organizational Recon

During **organizational** recon, you're going to be focused on collecting information about the organization.

This can include the people who work there (e.g. names, jobs, skills, etc.), the organizational structure, site locations, and business relationships.

#### Technical Recon

During **Technical** recon, you're going to be looking for systems such as public-facing websites, mail servers, remote access solutions, and any vendors or products in use, particularly defensive ones (e.g. web proxies, email gateways, firewalls, antivirus, EDR solutions, etc.).

### Data Gathering

Gathering data can be performed in a "passive" or "active" fashion.&#x20;

#### Passive Collection

This relies on third-party sources such as social media, Google, Shodan, etc.

Primarily consists data collection where you are not actively "touching" the target in any fashion.&#x20;

#### Active Collection

This one is obvious, this tactic involves directly touching various components of the target.

This could be as simple as visiting the target's website, port scanning IP ranges, etc.

{% hint style="info" %}
Note: Active recon is inherently riskier than passive, as it provides an origanzation with their first potential indication that they're being looked at.

:warning: To avoid OPSEC concerns and to protect it, be sure to perform active recon via a proxy or VPN service so you do not expose your public IP address.
{% endhint %}

## DNS Records

Domain Name System (DNS) records can provide a wealth of information regarding services that may be exposed to the Internet.

The fictional "target" we're going to attack in the lab is an organization called Cyberbotic. Their domain name is `cyberotic.io`.&#x20;

We can kick off by performing a simple lookup of any existing `A` records for this domain using the Domain Internet Groper (DIG):

```
dig cyberbotic.io
```

With this, we can obtain an IP address via `A` record that we can use for further discovery.&#x20;

By performing a `whois` on each IP address, we obtain a provider for the IP address and can see that it belongs to a third-party provider, Cloudflare.&#x20;

This helps us confirm whether the IP address is on-prem or cloud-provided as this will change our approach since providers like Amazon AWS and Microsoft Azure have different rules and require explicit permission to carry out security assessments on any assets hosted within their infrastructure.&#x20;

We can also perform a zone transfer to help discover DNS subdomains using `dig axfr <domain-name>` or a tool such as `dnscan.py`.&#x20;

## Weak Email Security (e.g. SPF, DMARC, & DKIM)

Certain protocols may allow you to be able to spoof emails to appear as if they are coming from their own domain.

[Spoofy](https://github.com/MattKeeley/Spoofy) is a Python script that can verify the email security of a given domain.

**Usage below:**

```
~/Spoofy$ pip3 install -r requirements.txt
~/Spoofy$ python3 spoofy.py -d cyberbotic.io -o stdout
[*] Domain: cyberbotic.io
[*] Is subdomain: False
[*] DNS Server: 1.1.1.1
[?] No SPF record found.
[?] No DMARC record found.
[+] Spoofing possible for cyberbotic.io
```

## Google Dorking

This is a fancy way of using Google's advanced search operators to leverage Open-Source Intelligence (OSINT) to find targeted, potentially sensitive information against the target.

`site:` Limit the search results to those from a specific website.

* (e.g.) `site:apple.com`

Returns some not so useful information but will return every page that Google has indexed for the `apple.com` domain.

`intitle:` find pages with a certain word in the title.

* (e.g.) `intitle:apple`&#x20;

And many many more, this is review for me and for many readers of this blog so I will be skipping covering the rest of these.

These dorks can be used to find interesting files, web applications, and information.
