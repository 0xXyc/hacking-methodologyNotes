---
description: 03/05/2024
cover: https://miro.medium.com/v2/resize:fit:622/1*6bLRa3i1QPSGchKEVFE9jw.jpeg
coverY: 0
---

# 🎃 Damn Vulnerable Web App (DVWA)

This is a simple attempt of getting better at web application security. A.K.A. my most disliked part of penetration/security testing. However, web apps are everywhere, and I want to get better at them!

In order to work on my methodology, I will be utilizing the Damn Vulnerable Web App (DVWA) available [here](https://github.com/digininja/DVWA) on GitHub for download. I will walk you through the installation/configuration guide as well as my methodology for pursuing this type of target in the form of a walkthrough.&#x20;

## Introduction

"Damn Vulnerable Web Application (DVWA) is a PHP/MySQL web application that is damn vulnerable. Its main goal is to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications and to aid both students & teachers to learn about web application security in a controlled class room environment.

The aim of DVWA is to **practice some of the most common web vulnerabilities**, with **various levels of difficulty**, with a simple straightforward interface. Please note, there are **both documented and undocumented vulnerabilities** with this software. This is intentional. You are encouraged to try and discover as many issues as possible.

### WARNING!

Damn Vulnerable Web Application is damn vulnerable! **Do not upload it to your hosting provider's public html folder or any Internet facing servers**, as they will be compromised. It is recommended using a virtual machine (such as [VirtualBox](https://www.virtualbox.org/) or [VMware](https://www.vmware.com/)), which is set to NAT networking mode. Inside a guest machine, you can download and install [XAMPP](https://www.apachefriends.org/) for the web server and database".

## Installation

I will be utilizing docker in order to setup my environment for ease-of-use and simplicity.

**Don't have docker installed?**

Check [this](https://miro.medium.com/v2/resize:fit:622/1\*6bLRa3i1QPSGchKEVFE9jw.jpeg) out.

**Clone the DVWA repo:**

```
git clone https://github.com/digininja/DVWA.git
```

Navigate to DVWA's working directory and run the following docker command:

<pre><code><strong>sudo docker compose up -d 
</strong></code></pre>

<mark style="color:yellow;">DVWA is now available</mark> at `http://localhost:4280`.

<figure><img src="../.gitbook/assets/image (186).png" alt=""><figcaption><p>DVWA Landing Page</p></figcaption></figure>

**Configure MYSQL database:**

```
cd /DVWA
cd config
cp config.inc.php.dist config.inc.php
nano config.inc.php

# Modify 'db_user' and 'db_password' if you wish. I changed mine to dvwa:password.
# Save and exit

sudo service mysql start
sudo mysql -u root -p
# Upon password prompt, press enter
create database dvwa;
create user 'admin'@'127.0.0.1'
create user 'admin'@'127.0.0.1' identified by 'password';
grant all privileges on dvwa.* to 'admin'@'127.0.0.1';
exit
```

**Apache2:**

```
sudo nano /etc/php/8.1/apache2/php.ini
# CTRL + W --> Search for fopen
allow_url_fopen = On
allow_url_include = On
# Save and exit
```

Configuration complete! <mark style="color:green;">Start hacking</mark>!

Note: <mark style="color:yellow;">If asked in the main menu, reload the database, and log back in</mark>.

For starters, we will begin with the lowest security difficulty within the **DVWA Security** section.

## Brute Force

### Difficulty: Low

