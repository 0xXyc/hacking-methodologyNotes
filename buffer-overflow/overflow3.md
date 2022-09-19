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
