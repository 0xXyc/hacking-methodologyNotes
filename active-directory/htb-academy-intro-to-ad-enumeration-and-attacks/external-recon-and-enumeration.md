# External Recon and Enumeration

## Introduction

<mark style="color:yellow;">Before performing any pentest, it can be highly beneficial to perform external reconnaissance on your target prior to the assessment. This can help you out in a multitude of ways:</mark>

* Validates information&#x20;
* Ensures that you are staying within the scope of assessment that is measured in the Rules of Engagement (RoE) document
* Look for information that is <mark style="color:yellow;">publicly accessible</mark> such as leaked credentials or usernames
  * <mark style="color:yellow;">This kind of information is extremely useful when it comes time to perform internal assessments!</mark>

### Why is this important?

* It allows us to "get a lay of the land" to ensure that we can provide an excellent and quality assessment to the client!

## What to look for During External Reconnaissance

"<mark style="color:yellow;">The why and what of external recon</mark>"

<mark style="color:yellow;">IP Space</mark>:&#x20;

Where will we be operating? What are the DNS records? Cloud presence, what are the in use net blocks? What kind of public-facing infrastructure can we see? Who are the hosting providers?

<mark style="color:yellow;">Domain Information</mark>:&#x20;

Mailservers, DNS, websites, VPN portals, etc.

* What kind of defenses are in place? Can we determine this?
  * SIEM, AV, IPS/IDS, etc.

<mark style="color:yellow;">Schema Formatting (Naming Conventions)</mark>:&#x20;

Can we discover the organization's email accounts, AD usernames, and password policies?&#x20;

* The goal here is to build a valid username list to test external-facing services&#x20;
  * Password spraying
  * Credential Stuffing
  * Bruteforcing
  * etc.

<mark style="color:yellow;">Data Disclosure</mark>:

Here, we are looking for publicly accessible files (pdf, ppt, docx, xlxs, etc.)

* Any information that can lead to a compromise is invaluable
* GitHub repo?
* Hard-coded credentials?
* Metadata in photos or PDF's?

<mark style="color:yellow;">Breach Data</mark>:

Are there any publicly released usernames, passwords, or other critical information that can help an attacker gain a foothold?

## Where are we Looking?

<mark style="color:yellow;">ASN/IP Registrars</mark>:

North and South America: IANA and ARIN

Europe: BGP Toolkit and RIPE

<mark style="color:yellow;">Domain Registrars & DNS</mark>:

Domaintools, PTRArchive, ICANN, and manual DNS record requests against the domain or against well known DNS servers (8.8.8.8)

<mark style="color:yellow;">Social Media</mark>:&#x20;

LinkedIn, Twitter, Facebook, region's major socials, news articles, and any other relevant information that Google can provide you with

<mark style="color:yellow;">Public-Facing Company Websites</mark>:&#x20;

The public website will often contain links for relevant information that is embedded

* <mark style="color:red;">News articles, documents, and the "About Us" and "Contact Us" pages are gold mines</mark>

<mark style="color:yellow;">Cloud & Dev Storage Spaces</mark>:

GitHub, AWS S3 Buckets, Azure Blog Storage Containers, Google searches using Dorks

<mark style="color:yellow;">Breach Data Sources</mark>:

<mark style="color:red;">HaveIBeenPwned</mark>- Determine if any corporate email accounts appear in public breach data

<mark style="color:red;">Dehashed</mark>- Search for corporate emails with cleartext credentials or hashes that we can crack offline

* You can use this information to curate wordlists and password lists to login to exposed login portals that use AD authentication

## Finding Address Spaces

BGP Toolkit: [https://bgp.he.net/](https://bgp.he.net/)

This is a fantastic resource for researching what address blocks are assigned to an organization

* Large companies will have lots of self-hosted infrastructure which means that they will have their own ASN
* Smaller companies will not have their own ASN typically
  * Keep in mind where someone else is hosting their infrastructure
    * Cloudflare, Google Cloud, AWS, or Azure

<mark style="color:yellow;">This is important because we need to be sure we are not testing infrastructure outside of our scope!</mark>

### DNS Information:

{% embed url="https://viewdns.info/" %}

{% embed url="https://whois.domaintools.com/" %}

## Example Situation

1. Check ASN/IP & Domain Data
2. ViewDNS Results
3. Hunt for Data Disclosure (i.e. files)
4. Hunt for E-mail address via "Dorking"&#x20;

### Check ASN/IP & Domain Data

<mark style="color:yellow;">Let's utilize the tactics above on the inlanefreight.com domain!</mark>

{% embed url="https://bgp.he.net/dns/inlanefreight.com" %}
BGP Toolkit
{% endembed %}

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption><p>Flag obtained in TXT record</p></figcaption></figure>

* We can see that we have a decent sized attack surface
* Nameservers
* Mail exchanges

### ViewDNS Results

{% embed url="https://viewdns.info/reverseip/?host=inlanefreight.com&t=1" %}

<figure><img src="../../.gitbook/assets/image (3) (6).png" alt=""><figcaption></figcaption></figure>

* ViewDNS should be used to compare the results of BGP
* Now, we can take the nameservers obtained from BGP and perform Nslookup on them

### Nslookup (CLI tool)

<figure><img src="../../.gitbook/assets/image (5) (7).png" alt=""><figcaption></figcaption></figure>

### Hunting For Files

* We can now Google Dork to look for specific files that can be associated with a particular domain
* Domain: inlanefreight.com

```
filetype:pdf inurl:inlanefreight.com
```

<figure><img src="../../.gitbook/assets/image (1) (7).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.inlanefreight.com/wp-content/uploads/2020/09/goals.pdf" %}
Embedded
{% endembed %}

### Hunting for E-mail Addresses

* You can also Google "Dork" for E-mail addresses!

Domain: inlanefreight.com

```
intext:"@inlanefreight.com" inurl:inlanefreight.com
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* Upon visiting the "Contact" web page, we are granted with different email addresses
* Pay attention to the naming convention, is it first.last, last.first, first initial.lastname?
* These are all very important to know so you can make an accurate guess at the naming convention used inside the domain

In our case, it is first.lastname!&#x20;

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Credential Hunting

* Dehashed is an excellent tool for hunting cleartext credentials and password hashes in breach data
* You can choose to use a script via the API or directly from the site

{% embed url="https://dehashed.com/" %}
DeHashed Official website
{% endembed %}

{% embed url="https://github.com/sm00v/Dehashed" %}
GitHub Repository
{% endembed %}

