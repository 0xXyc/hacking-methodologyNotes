---
description: 11/21/22
---

# Cobalt Strike C2

## <mark style="color:green;">Starting C2 Server</mark>

```
tmux

cd /cobaltstrike

sudo ./teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile
```

* Then start the <mark style="color:yellow;">Cobalt Strike Client</mark>
* Configure your client as needed and you will land in the Cobalt Strike Client

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:green;">Listener Management</mark>

* The next step is to setup "listeners", these will listen for incoming connections from Cobalt Strike's Beacons

There are two types of listeners:

* Egress
* Peer-to-Peer

### Egress Listeners

* An egress listener is one that <mark style="color:yellow;">allows Beacon to communicate to outside of the target network to our team server</mark>
* The default egress listeners are <mark style="color:yellow;">HTTP/S & DNS</mark> where the Beacon will encapsulate all C2 traffic over these protocols

Managing Listeners (<mark style="color:yellow;">add, edit, remove, or restart listeners</mark>):

Cobalt Strike > Listeners or click on the headphones icon

### HTTP

* The HTTP listener allows Beacon to send and receive C2 messages over HTTP GET and/or POST requests
* To create a new HTTP listener, click add
*
