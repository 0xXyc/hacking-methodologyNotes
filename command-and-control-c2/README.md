---
description: 08/31/2025
---

# Command & Control (C2)

## Motivation

We love command & Control servers.&#x20;

Here, we are going to be diving into Cobalt Strike.

## Listeners

There are two main types of listeners:

1. egress
2. peer-to-peer

## Egress Listeners

An egress listener is one that allows Beacon to communicate outside of the target network to our team server.

The default egress listener types within Cobalt Strike are HTTP/S and DNS, where Beacon will encapsulate C2 traffic over these protocols accordingly.

### Managing Listeners

In order to manage your listeners, you can go to:

Cobalt Strike -> Listeners (or alternatively, click on the headphone icon).

This will open up the Listeners tab, where you can add, edit, remove, and restart listeners.

## HTTP

The HTTP listener allows Beacon to both send and receive C2 messages via HTTP `GET` and `POST` requests.

In order to create a new HTTP listener, click **add**.

Make sure that **Beacon HTTP** is selected as payload type and give the listener a name.

* This name will be used throughout the Cobalt Strike CLI commands, so be sure to use one you can remember.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### OPSEC Consideration

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Click the + button next to the HTTP Hosts box. Here, you will provide the IP address/domain names that the Beacon payload will call back to.&#x20;

By default, it will auto-populate the IP address of the Attacker Linux VM, but it is more realistic to use proper DNS names.

&#x20;After naming and selecting save, you will see a message appear in the team server and port 80 will now be listening.

## Peer-to-Peer (P2P)

These listeners differ from egress listeners because they don't communicate with the team server directly. Instead, P2P listeners are designed to chain multiple Beacons together in parent/child relationships. The primary reasons for doing this are:

1. To reduce the number of hosts talking out to your team server. This is because the higher the traffic volume, the more likely it is to get spotted.&#x20;
2. To run Beacon on machines that aren't able to talk outside of the network (e.g. firewall rules and network segmentation).&#x20;

The two P2P listener types in Cobalt Strike are Server Message block (SMB) and raw Transmission Control Protocol (TCP — "three-way handshake").&#x20;

It is important ot understand that these protocols do not ever leave the target network and instead a child SMB/TCP Beacon will be linked to an egress HTTP/DNS Beacon, and the traffic from the child is then sent to the parent, which then in turn sends it off to the team server.

## SMB

SMB listeners are very simple and only have a single option, this is the named pipe name.&#x20;

The default is `msagent_##` (where `##` is random hexadecimal), you can specify whatever you want.

A Beacon SMB payload will start a new named pipe server with this name and listen for an incoming connection.

This named pipe is available both locally and remotely.

#### OPSEC Consideration

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### Listing out all currently listening pipes

```
PS C:\Users\Attacker> ls \\.\pipe\
```

From that list of listening pipes on the machine, we can grab one of those to make it look like it belongs and is benign, and replace the last four digits (or something like that) within the **pipename (C2)** option.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Above, I replaced the last four characters within the Pipe to `1337`, where the original was:

```
TSVCPIPE-3e6a911c-e5ec-4f36-b351-342196bc181a
```
