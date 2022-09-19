---
description: https://tryhackme.com/room/bufferoverflowprep (oscp.exe)
---

# OVERFLOW2

## Methodology

1\. Spiking or Playing with the program

2\. Fuzzing

3\. Controlling the EIP

4\. Finding Bad Characters

5\. Finding a Jump Point

6\. Generating a Payload

7\. "NOP Sledding"

8\. Exploitation

## Spiking or Playing

* Opened up the vulnerable program in Immunity
* Ran the program
* Noticed in Command Prompt that it is running a server on 1337
* Connected to the program via Netcat to see the inner workings of the program

## Mona Configuration

Place the following into the debugger window of Immunity:

```
!mona config -set workingfolder c:\mona\%p
```

## Fuzzing

* Utilize the <mark style="color:yellow;">fuzzer.py</mark> script to fuzz the program in an attempt to see where we can make the program break
* Be sure to note when the program crashes (what bytes)
* Run the command and configure as needed (Change IP address and prefix to OVERFLOW2)

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Running the fuzzer:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">We note that it crashed at 700 bytes</mark>

## Crash Replication and Controlling EIP

* We will now utilize <mark style="color:yellow;">exploit.py</mark>

Contents of exploit.py:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### msf\_pattern\_create

* We now need to generate a program that is a length of 400 bytes longer than the string that crashed the program
* This is because we need to still make room for the payload in memory
* <mark style="color:yellow;">1100</mark> bytes is what we will use

Use the following command to generate your pattern:

```
msf-pattern_create -l 1100
```

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* Copy/paste the string into the <mark style="color:yellow;">payload</mark> section of <mark style="color:yellow;">exploit.py</mark>
* Return to Immunity and restart the program and run it again
* Now that the program is running, it is time to run our new exploit
* Upon executing, we should see the following in Immunity debugger:

<mark style="color:yellow;">Access violation when executing \[37784136]</mark>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* Notice that we are making an access violation occur on the EIP <mark style="color:yellow;">(37784136)</mark>

### Find the Distance from EIP







































































































