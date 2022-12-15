# Address Space Location Randomization (ASLR)

## Memory Protection Technique

* This technology is found in many Operating Systems and devices
* It works by randomizing the memory address of a process in the stack/heap

### Disabling & Enabling ASLR (Linux)

Disabling:

```
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

Enabling:

```
echo 2 | sudo tee /proc/sys/kernel/randomize_va_space
```
