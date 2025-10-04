---
description: 06/09/2024
cover: ../../.gitbook/assets/Screenshot 2024-06-09 at 2.01.36 PM.png
coverY: 137
---

# Creating Your own VPN

## Introduction

<mark style="color:yellow;">**Why?**</mark>

### <mark style="color:yellow;">**Are you concerned about your privacy?**</mark>

<mark style="color:red;">Please, do not fall under the group of people who "have nothing to hide".</mark>&#x20;

Privacy to YOUR information, data, PII, etc. should be a given right and shouldn't be something you have to worry about. So, I encourage all who is reading this to take control of things and variables that you can control and make it your own when you can.

In recent years, there's so much talk about why you should use VPN's and why you shouldn't.

The goal of this write up is to show you how easily you can achieve the comfort of a VPN without having to worry about the "politics" of strange, and somewhat suspicious VPN provider antics.&#x20;

### <mark style="color:yellow;">**Give me some examples!**</mark>

Okay, so VPN providers in the past have made various claims to "full anonymity" as well as a "strict no-log claim" which means that the activities you perform online while connected to their service are completely end-to-end (E2E) encrypted in addition to their users being entitled to a no-logging policy.

Unfortunately, VPN providers have went against their users in an act to combat cyber terrorism and abuse. Which means that even if you aren't doing anything malicious, you may have been caught in that crossfire without even noticing.&#x20;

This is why if you control your own VPN endpoint/solution, you control the entire ecosystem for your E2E solution. Meaning, YOU are in charge of your logs, YOU are in charge of the implementation etc.

This will grant you an almost immediate "peace of mind" once you implement this solution within your workflow.

The good thing? It doesn't take much time to implement this solution on your own or require as much technical knowledge as you think it might. This is all due to amazing online services and open-source technologies.

### <mark style="color:red;">**Disclaimer**</mark>

Unfortunately, security often times comes at a cost of monetary, time, and a loss of convenience (amount varies).&#x20;

This is because it takes money to implement procedures and technologies yourself, but frees you of any worries you may have with other politics or policies you may not be familiar with.&#x20;

This can even come in the form of you not even being aware of them in the first place.&#x20;

When you "agree" to the End User License Agreement (EULA) -- this is the large legal contract that you quickly click "agree" on without reading anything; and I can't blame you for not reading it.

## Getting Into It

**This is going to require a few different technologies.**

1. <mark style="color:green;">**VPS Solution**</mark>
2. <mark style="color:green;">**Cryptocurrency (optional, but highly recommended)**</mark>
3. <mark style="color:green;">**OpenVPN or WireGuard (OpenVPN is superior -- secure, but slightly more complicated)**</mark>

You have a choice to follow my recommendation/technologies/solutions that fit this same methodology. However, I highly recommend you utilize an underlying VPS solution that enables payments with cryptocurrency. With that said, I will only be using my recommendation in this write up.

To add a layer of security/complexity to your operation, I believe you should utilize a VPS solution that takes payments in the form of cryptocurrency to further enhance your privacy capabilities. Not only will this not directly "tie your name" to the server (thanks to the increased anonymity that blockchain technology naturally provides). Keep in mind, this greatly&#x20;

> **🚨 Keep in mind, this solution greatly depends on the cryptocurrency platform you are using, the type of crypto you are using (Monero and Bitcoin are solid "anonymous" choices), and the credibility of your VPS solution.**

## Purchase Some Crypto

Go ahead and purchase some crypto, Ethereum, Monero, or Bitcoin is recommended.

Be sure that you have access to this crypto and have the ability to send it to an external source.

This is so that you can send your crypto from your wallet to the BitLaunch's wallet so they can receive your crypto as a form of payment.&#x20;

Once that's complete, you can move onto creating a VPS that is funded by crypto!

## VPS Solution

{% embed url="https://app.bitlaunch.io" %}
VPS Solution that allows payments via crypto
{% endembed %}

**This is a solid VPS solution that I've used in the past.**

Once you find a server, go ahead and select the cheapest option and select apps, and select OpenVPN.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="352"><figcaption><p>OpenVPN application in BitLaunch</p></figcaption></figure>

This will install OpenVPN in an automated fashion on top of an Ubuntu instance.

I also recommend disabling password authentication by default. This will greatly enhance the overall security of your VPS.&#x20;

**Okay, but how do we authenticate?**

I'm glad you asked, we will be using public key infrastructure (SSH keys) rather than a direct SSH password for authentication. This will boost the security of your server greatly because it's a whole lot easier to bruteforce a password than it is to mishandle a key, leading to hijacking, and loss of your server or surveillance that you do not want. Especially in this infrastructure.

Keep on reading, I will be providing a solution for applying "security best practices" to your VPS; keeping it secure from hackers.

### Generate a Public/Private key Pair

```
ssh-keygen -t rsa -b 4096
```

Follow through the key generation process, giving it a unique name (default is `id_rsa`).

You can leave the passwords and other data blank if you wish, this is entirely up to you.&#x20;

Once you have done that, you will have two files, a public key (`id_rsa.pub`) and a private key (`id_rsa`). You will be supplying BitLaunch the public key.

> **🚨 Never reveal, transfer, or share your private key. If you lose, this, you will be compromised.**
>
> **Only share your public key.**

### Select Authentication Method (Inside BitLaunch)

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption><p>SSH Keys (Recommended) Solution</p></figcaption></figure>

Add your public key data here, you can simply just copy/paste.

Next, launch the server and wait a few minutes for your server to be created.

### Authenticating to Your new VPS

```
ssh -i id_rsa root@<VPS_IP_HERE>
```

Boom, we're in!

## Accessing OpenVPN web Interface

**Change your OpenVPN admin password:**

**With root access, we can do this!**

> <mark style="color:red;">🚨</mark> <mark style="color:red;"></mark><mark style="color:red;">**Please make this extremely, obnoxiously secure. For obvious reasons.**</mark>

```
passwd openvpn
# Enter new password here, it will prompt twice
```

We can now access the UI with credentials, `openvpn:<newly_created_password>`.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption><p>OpenVPN Admin Login</p></figcaption></figure>

You will now see that you can access OpenVPN's web UI directly the VPS' IP followed by port 943.

**It will prompt you that this certificate is not trusted, not to worry, ignore this and continue.**

Now, we can authenticate to the `/admin` interface using the username and password generated at the initial prompt when you SSh'd in.

Now, create a new user.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Creating a new user</p></figcaption></figure>

Add a new user, username, select "Allow Auto-login", and select more settings if desired.

This will allow you to create a password.

### Once Created

Navigate to the normal user web interface for OpenVPN.

Authenticate with your new user and password.

It will then take you to a link where you can download an OpenVPN client.

This should automatically take your configuration and place it in your OpenVPN client, however, with mine it didn't, so I had to import a new configuration.

Once I did this, I was able to use my brand new VPS as a completely personalized VPN solution!

## But Wait, There's More

This is by far the most important part.

You're not done yet.&#x20;

Remember those "security best practices" I was hinting at before? Yes, it's time for those. Conveniently, I have wrapped all of this in a nice little script for you all.

{% embed url="https://github.com/0xXyc/VPS-Lock-Figuration" %}
Check this out
{% endembed %}

This is a nice little BASH script that you can import and execute on your VPS. It will automatically apply all up-to-date best practices on your VPS.

Please follow the steps accordingly to prevent a total lockout to your new VPS.

**It will perform the following:**

1. **Update system packages**
2. **Create a new user with sudo privileges.**
3. **Set up a firewall using UFW to allow only inbound SSH on port 13337 (by default) .**
4. **Disable root login and password authentication via SSH.**
5. **Install and configure Fail2Ban.**
6. **Enable automatic security updates.**

> <mark style="color:yellow;">**🚨 Once this script has been ran, do NOT disconnect from your VPS, you will still need momentary access to complete the setup.**</mark>

With your newly created user, this will mean you must access your VPS on a new port as well as a new user now that root and password authentication is disabled.

**We can do that by the following:**

Create a new directory on your local host with the name of your new user.

Create a new public/private key pair (same as before).

Copy/paste the contents of the public key into a file in `/.ssh` in your new user's directory on the VPS, not on your host.

**Create a new file inside of `/.ssh`:**

```
touch authorized_keys

nano authorized_keys

# Add contents of copied public key in here
```

### Connecting Back to Your VPS w/ the new User

**On your local system:**

```
ssh -i id_rsa -p <new_ssh_port_here> <new_user>@<vps_IP_Here>
```

Boom, you're in!

Enjoy your new VPS/VPN solution! You control everything here!

### Ensuring you are Connected to Your new VPN

To make sure that your traffic is being encrypted, we can do the following:

**Check IP address:**

```
curl ifconfig.me
# This should return an IP address that is different than your original
```

**Run Wireshark while connected to the OpenVPN client:**

You should see OpenVPN communications occurring constantly as well as encrypted data in the TCP streams.&#x20;

## Things to be Mindful of

Just remember, this is your only form of protection and layer of security. If your VPS gets popped, your original IP address could get leaked.

{% hint style="info" %}
Be sure your OpenVPN is running the latest version. Check out the next page to verify the latest version is running.
{% endhint %}
