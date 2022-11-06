---
description: 10.10.0.181
---

# WEB-1

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4d:3f:9a:c0:6b:0a:ce:49:aa:aa:00:5b:03:2e:28:75 (RSA)
|   256 3b:e4:01:84:86:db:26:46:54:1c:4f:2c:20:7c:c7:ef (ECDSA)
|_  256 c9:73:79:9b:e5:99:ae:dc:10:65:e3:e7:31:f5:f5:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
No UDP ports are open
```

Notes:

* 22/SSH -- OpenSSH 8.2p1
* 80/HTTP -- Apache 2.4.41

## Enumeration

### Port 80 - HTTP (Apache 2.4.41)

* Checked source code -- /get\_repos.php
  * [https://10.10.0.103/brandon/website-dev](https://10.10.0.103/brandon/website-dev)
  * This took me to a GitLab webpage
  * [https://10.10.0.103/brandon/website-dev/-/blob/main/get\_repos.php](https://10.10.0.103/brandon/website-dev/-/blob/main/get\_repos.php)

Contents of get\_repos.php

{% code overflow="wrap" %}
```
<?php
$serverName = "";
$connectionInfo = array( "Database"=>"", "UID"=>"", "PWD"=>"");
$conn = sqlsrv_connect( $serverName, $connectionInfo);
if ($conn){
    echo "Connected to database!\n\n";
}else{
    die(print_r(sqlsrv_errors(), true));
}
$sql = "SELECT * FROM repos";
$stmt = sqlsrv_query( $conn, $sql );
if( $stmt === false) {
    die( print_r( sqlsrv_errors(), true) );
}
while( $row = sqlsrv_fetch_array( $stmt, SQLSRV_FETCH_ASSOC) ) {
    echo "<br>";
    print_r($row, false);
}
sqlsrv_free_stmt( $stmt);
?>
```
{% endcode %}

* Opened README.md nothing is in it
* Ran directory bruteforce -- nothing
* Ran vhosts -- Nothing

<figure><img src="../../../.gitbook/assets/image (10) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

* How can we leverage what we have?
* Upon going back to the repo, I go to check the version history. I seem to find a password here

{% embed url="https://10.10.0.103/brandon/website-dev/-/commit/1d331bf3998e3348a7fdf4bf9a42bb8c5707caec" %}

```
$connectionInfo = array( "Database"=>"gitlab", "UID"=>"sa", "PWD"=>"VeryVeryVerySecure1!");
```

* Attempted to SSH into the machine as an easy win with `ssh brandon@10.10.0.103`
* However, this did not work as we need a key

## Exploitation

### SSH Private Key Reusage

* Brandon's key that was obtained on WEB2 has been reused on WEB1
*



## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
