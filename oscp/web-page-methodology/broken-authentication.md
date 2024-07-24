# Broken Authentication

"How can we authenticate to the device or how can we get in?"

* When you gain access, this is broken authentication

## Well Known Service?

1. Default Credentials
2. Weak Credentials
3. Admin Admin?

It is important to understand that these are fundamentally authentication issues in general.

Tips to prevent broken authentication:

* <mark style="color:yellow;">MFA</mark>
* <mark style="color:yellow;">Do not use weak credentials</mark>
* <mark style="color:yellow;">Rotate user ID's</mark>
* <mark style="color:yellow;">Rate limiting or 2captcha to prevent automated, botlike, or brute forcing behavior</mark>

## Testing for Broken Authentication

You are presented with a login page...

<figure><img src="../../.gitbook/assets/image (2) (4) (2).png" alt=""><figcaption></figcaption></figure>

Perform some username enumeration based on attempted login errors.

If you provide a valid email, does it change the behavior of the site?

Look at all of the features!

Be sure to capture all requests with Burp!

