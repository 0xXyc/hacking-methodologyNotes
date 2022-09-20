---
description: Dummy shell to fully-interactive shell upgrade
---

# 😎 Fully-Interactive Shell Upgrade

## You just popped a shell...

Python:

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

If Python3 is available:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Example syntax:

```
/var/www/admin$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```

* The shell will look exactly the same or it may appear slightly differently

However, to gain full functionality do the following:

<mark style="color:yellow;">Press CTRL+Z</mark>

```
 stty raw -echo; fg
```

* <mark style="color:yellow;">Press enter a few times after this</mark>
* <mark style="color:yellow;">export TERM=xterm-256color</mark>
* Now, you will have a fully-interactive shell
* This includes tab/auto-complete
* CTLR+C
* And more
