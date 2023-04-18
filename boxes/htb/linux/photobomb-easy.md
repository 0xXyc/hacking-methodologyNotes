# Photobomb (Easy)

<figure><img src="../../../.gitbook/assets/Photobomb.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e22473bbfbdf5cb520b66876748ab58d (RSA)
|   256 04e3ac6e184e1b7effac4fe39dd21bae (ECDSA)
|_  256 20e05d8cba71f08c3a1819f24011d29e (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Photobomb
|_http-favicon: Unknown favicon MD5: 622B9ED3F0195B2D1811DF6F278518C2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

Port 80 is open, this will most likely be the entry point for this machine.&#x20;

* <mark style="color:yellow;">"Credentials are in your welcome pack"? -- View visual inspection</mark>
* Added photobomb.htb to /etc/hosts

Port 22 is open, can we steal SSH keys or produce them to establish a better shell?

## Enumeration

### Port 80 - HTTP (nginx 1.18.0)

#### Visual Inspection:

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

* Check page source
* Credentials are stored here, but we are presented with a login field

#### Page Source:

photobomb.js

{% code overflow="wrap" %}
```
function init() {
  // Jameson: pre-populate creds for tech support as they keep forgetting them and emailing me
  if (document.cookie.match(/^(.*;)?\s*isPhotoBombTechSupport\s*=\s*[^;]+(.*)?$/)) {
    document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');
  }
}
window.onload = init;
```
{% endcode %}

* We are given credentials here!

Navigate to the following in the URL bar:

```
http://pH0t0:b0Mb!@photobomb.htb/printer
```

* It will ask if you want to authenticate with these credentials, select yes
* We now have access to /printer

#### Dirsearch:&#x20;

```
dirsearch -u photobomb.htb
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

#### /printer:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* We are met with a page that allows you to download images with different file types and dimensions
* Let's download an image and capture the request in burp

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Let's see if we can exploit the fields in this request and get some strange behavior.

Attempt LFI vulnerability:

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

* However, we get a response that says invalid photo
* This makes me know that there are some kind of validation checking techniques in play here
* How can we get around this?

#### Command Injection:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

The semicolon allows us to pass through additional commands. When I was able to check how long it takes for the request to be processed by the server with the sleep command.&#x20;

With semicolon: 3.3 milliseconds

With semicolon and `sleep 5` command (Be sure to URL encode this as seen in the screenshot): 8.4 seconds&#x20;

* This confirms that we have successful command injection on our target
* Time for exploitation

## Exploitation

### Command Injection

The injection point is found in <mark style="color:yellow;">http://photobomb.htb/printer</mark> upon downloading an image and adding a semicolon within one of the fields. Immediately after the semicolon, place your URL-Encoded reverse shell here.&#x20;



## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
