# Enumerating SMB

## smbclient

Anonymously list a SMB share:

```
smbclient -L \\\\10.10.10.1\\
```

Anonymously authenticate to an SMB share:

```
smbclient \\\\10.10.10.1\\ADMIN$

smbclient \\\\10.10.10.1\\IPC$
```
