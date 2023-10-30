---
description: 10/27/2023
cover: https://i.gifer.com/EgUx.gif
coverY: 167
---

# Methodologies

## Reversing-101

**Do this **<mark style="color:yellow;">**everytime**</mark>** you begin reversing (will be adding to this) :**

1. <mark style="color:yellow;">**Hash**</mark>** file for records:**

```
md5sum ./binary
sha256sum ./binary
```

2. **View the **<mark style="color:yellow;">**raw file**</mark>**:**

```
hexdump -C ./binary | head -10
man ascii
```

3. <mark style="color:yellow;">**Parse bytes**</mark>** and show ONLY ASCII:**

```
strings <binary>
```

4. **Obtain **<mark style="color:yellow;">**file signature**</mark>**:**

```
file <binary>
```

5. **Document findings** with screenshots, theory, and context in an attempt to further "paint the picture"
6. <mark style="color:yellow;">**Obtain symbols for imported functions**</mark>** of the binary:**

```
readelf -W --dyn-sym ./binary
```

7. **Utilize `objdump` to **<mark style="color:yellow;">**view disassembly and examine specific ELF sections**</mark>**:**

<pre><code><strong>objdump -s -j .rodata ./binary
</strong></code></pre>

8. <mark style="color:yellow;">**Check symbols**</mark>** in the binary using `nm`:**

```
nm <binary>
```

**If desired, you can **<mark style="color:yellow;">**strip the binary of symbols manually**</mark>** with `strip`:**

```
cp <binary-symbols> <binary-stripped>
strip <binary-stripped>
file <binary-stripped>
```

9. Throw binary inside of <mark style="color:yellow;">static analysis</mark> tool of choice, Ghidra or IDA

## Necessary Tasks

* Check strings&#x20;
* Check symbols
* Check for library imports if defined
* If the architecture is known, look at the memory layout and look at what registers are used for what
* Figure our how the architecture initializes the code e.g. vector table if symbols are not present
* View the spec sheets of what you are reversing if possible
* Look for unique data such as ID's, IP's, or MACs

## Resources :books:

{% embed url="https://medium.com/@Asm0d3us/1-crackmes-one-beginner-friendly-reversing-challenges-6df94ea6b29d" %}

{% embed url="https://osandamalith.com/2019/02/11/linux-reverse-engineering-ctfs-for-beginners/" %}

{% embed url="https://wrongbaud.github.io/posts/ghidra-training/" %}

{% embed url="https://medium.com/swlh/intro-to-reverse-engineering-45b38370384" %}

{% embed url="https://medium.com/swlh/intro-to-reverse-engineering-part-2-4087a70104e9" %}

{% embed url="https://medium.com/swlh/intro-to-reverse-engineering-45b38370384/" %}

## Videos

{% embed url="https://www.youtube.com/watch?v=ld2Y_5e4yZ4" %}

{% embed url="https://www.youtube.com/watch?v=fTGTnrgjuGA" %}







## Tools

{% embed url="https://dogbolt.org/" %}

{% embed url="https://godbolt.org/" %}
