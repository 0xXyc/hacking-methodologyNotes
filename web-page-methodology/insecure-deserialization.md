# Insecure Deserialization

## Understanding Serialization and Deserialization

### Serialization

* Convert an object into a format that can be placed on a disk
* To be sent over a network
* Serialized data can be YAML, Binary, XML, JSON, etc.

### Deserialization

* <mark style="color:yellow;">This is the opposite process of serialization</mark>
* In other words, you are taking the serialized data and deserializing it!

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## Insecure Deserialization

<mark style="color:yellow;">The nature of an insecure deserialization attack is when you take a malicious piece of code or a payload, serialize it, and introduce it to a web application.</mark>&#x20;

<mark style="color:yellow;">Upon introduction, the web application will theoretically begin the deserialization process and in return, execute the malicious code.</mark>

* Web apps use deserialization and deserialization very often
* These are very hard to find

## Ysoserial GitHub Insecure Deserialization PoC Payload Generator

{% embed url="https://github.com/frohoff/ysoserial" %}
