---
description: 10/25/2025
---

# Setting up Guest OS (VM) to Emulate x86/64 Userland Environment

For those who have a Mac running Apple Silicon and want to perform vulnerability research on x86/64 architecture without having to switch devices.

```bash
# 1) Install required packages on the ARM64 guest
sudo apt update
sudo apt install qemu-user-static binfmt-support debootstrap git

# 2) Create an amd64 chroot (example: Debian bookworm)
sudo mkdir -p /srv/chroot/amd64
sudo debootstrap --arch=amd64 bookworm /srv/chroot/amd64 http://deb.debian.org/debian

# 3) Copy the qemu static binary so amd64 ELF can run inside chroot
sudo cp /usr/bin/qemu-x86_64-static /srv/chroot/amd64/usr/bin/

# 4) Mount pseudo-filesystems and enter the chroot
for fs in proc sys dev dev/pts; do sudo mount --bind /$fs /srv/chroot/amd64/$fs; done
sudo chroot /srv/chroot/amd64 /bin/bash

# now you're inside an amd64 userspace. Verify:
# inside chroot:
uname -m        # may still show aarch64 for the kernel, but userland is amd64
ldd --version   # shows the amd64 glibc inside the chroot
```
