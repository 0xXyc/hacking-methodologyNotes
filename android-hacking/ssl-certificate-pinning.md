---
description: Preventing MITM's one pin at a time
---

# 📍 SSL Certificate Pinning

## What is Certpinning?

<mark style="color:yellow;">Certificate pinning is a security technique that was originally devised as a means to mitigate Man-in-The-Middle (MiTM) attacks.</mark> The application accepts only authorized or "pinned" certificates for authentication of client-server connections.

<mark style="color:yellow;">Any attempted secure connection requests utilizing non-pinned certificates are refused.</mark>

<mark style="color:red;">This technique is usually only found in very expensive, lucrative applications or other pieces of technology because of how expensive it is to implement this technique.</mark>&#x20;

## How does it work?

<mark style="color:yellow;">The idea is actually very simple; it is accomplished by correlating a host, from which connections will be sent, with the predetermined certificate.</mark>

Once the association has been established between the host and the certificate, the relationship is formed, and the certificate has been pinned to the host.

<mark style="color:yellow;">Pinning adds an extra layer of security by making it more difficult for a would-be attacker to compromise a pin by placing themselves in the middle of the communication (hence Man-in-The-Middle).</mark> This can be especially important during the early development phase. The pinning process can also take place once an application first attempts to connect. This is known as "key continuity".

## Common Mistakes Made by Developers During Development

<mark style="color:yellow;">Application developers are free to choose what certificate chain is pinned.</mark>

At minimum, one certificate in the chain of trust should be controlled solely by the organization that owns the application. <mark style="color:yellow;">However, an often repeated mistake is to only pin the root certificate in a chain.</mark>

<mark style="color:yellow;">Why is this so bad?</mark> This over-simplification of the process will leave a vulnerability in which a malicious adversary can gain access by leveraging a certificate obtained from the same Certificate Authority (CA).

## Security Recommendations

Developers should HIGHLY consider adding more than one or all certificates in the chain including:

#### <mark style="color:green;">Root Certificates</mark>

A root certificate is issued by a trusted CA based on a defined certificate validation method with public/private key pairing. It is very carefully monitored and protected by the root CA that issues them.

#### <mark style="color:green;">Intermediate Certificates</mark>

Intermediate certificates serves as an intermediary between the leaf and root certificates. While there must be at least one intermediate cert in a chain, it is also possible to deploy multiple intermediates within a single chain.&#x20;

It is possible to perform pinning with only the intermediate certificates but it is not recommended because you place all of your trust in the intermediate CA.

#### <mark style="color:green;">Leaf Certificates</mark>

Leaf certificates are the highest-level certificate in a chain. The pinning of a leaf certificate assures a certificate match. These are also known as end-user or end-entity certificates.

## Problems with Certpinning

* Resolution of compromised keys
* Very expensive to maintain
* Lack of cryptographic agility; cryptographic changes happen way more often than we think. Since this is another issue, it complicates everything further
* We also will see how if you own the device/target you are attacking, the certpinning defense can almost ALWAYS be bypassed with Frida!

#### "Hook.js"

```javascript
setTimeout(function(){
    Java.perform(function (){
    	console.log("");
	    console.log("[.] Cert Pinning Bypass/Re-Pinning");

	    var CertificateFactory = Java.use("java.security.cert.CertificateFactory");
	    var FileInputStream = Java.use("java.io.FileInputStream");
	    var BufferedInputStream = Java.use("java.io.BufferedInputStream");
	    var X509Certificate = Java.use("java.security.cert.X509Certificate");
	    var KeyStore = Java.use("java.security.KeyStore");
	    var TrustManagerFactory = Java.use("javax.net.ssl.TrustManagerFactory");
	    var SSLContext = Java.use("javax.net.ssl.SSLContext");

	    // Load CAs from an InputStream
	    console.log("[+] Loading our CA...")
	    var cf = CertificateFactory.getInstance("X.509");
	    
	    try {
	    	var fileInputStream = 
FileInputStream.$new("/data/local/tmp/cert-der.crt");
	    }
	    catch(err) {
	    	console.log("[o] " + err);
	    }
	    
	    var bufferedInputStream = BufferedInputStream.$new(fileInputStream);
	  	var ca = cf.generateCertificate(bufferedInputStream);
	    bufferedInputStream.close();

		var certInfo = Java.cast(ca, X509Certificate);
	    console.log("[o] Our CA Info: " + certInfo.getSubjectDN());

	    // Create a KeyStore containing our trusted CAs
	    console.log("[+] Creating a KeyStore for our CA...");
	    var keyStoreType = KeyStore.getDefaultType();
	    var keyStore = KeyStore.getInstance(keyStoreType);
	    keyStore.load(null, null);
	    keyStore.setCertificateEntry("ca", ca);
	    
	    // Create a TrustManager that trusts the CAs in our KeyStore
	    console.log("[+] Creating a TrustManager that trusts the CA in our KeyStore...");
	    var tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
	    var tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
	    tmf.init(keyStore);
	    console.log("[+] Our TrustManager is ready...");

	    console.log("[+] Hijacking SSLContext methods now...")
	    console.log("[-] Waiting for the app to invoke SSLContext.init()...")

	   	SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", 
"java.security.SecureRandom").implementation = function(a,b,c) {
	   		console.log("[o] App invoked javax.net.ssl.SSLContext.init...");
	   		SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", 
"java.security.SecureRandom").call(this, a, tmf.getTrustManagers(), c);
	   		console.log("[+] SSLContext initialized with our custom TrustManager!");
	   	}
    });
},0);

```
