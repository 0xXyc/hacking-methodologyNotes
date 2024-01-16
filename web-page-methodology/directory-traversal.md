---
description: Dot-Dot-Slash!
---

# Directory Traversal

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md" %}

## PHP

Do you have a file parameter in the URL bar?

* <mark style="color:yellow;">?file=</mark>

```
dev.holo.live/img.php?file=../../../../../../../../../../etc/passwd
```

This technique should be used with extreme patience. It is important for the attacker to ensure nothing is missed while traversing directories (enumerating).&#x20;

Upon success this can ultimately lead to Local File Inclusion (LFI) which means we can expose/reveal files within the file system on the target server.

* Does the site have a robots.txt?&#x20;
  * Check the contents and include the file
  * This could be a configuration page with sensitive information
* For example, wp-config.php contains the database's credentials!

```
dev.holo.live/img.php?file=../../../../../../../../../../var/www/wordpress/wp-config.php
```
