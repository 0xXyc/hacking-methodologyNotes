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

<figure><img src="../.gitbook/assets/image (4) (2) (3).png" alt=""><figcaption></figcaption></figure>

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

* The <mark style="color:yellow;">HTTP listener allows Beacon to send and receive C2 messages over HTTP GET and/or POST requests</mark>
* To create a new HTTP listener, click add, and select <mark style="color:yellow;">Beacon HTTP</mark> for the payload

<figure><img src="../.gitbook/assets/image (1) (1) (1) (4) (2).png" alt=""><figcaption></figcaption></figure>

* Once done, select save and you will see "Started Listener"

<figure><img src="../.gitbook/assets/image (3) (2) (2).png" alt=""><figcaption></figcaption></figure>

* Running a <mark style="color:red;">`sudo ss -lntp`</mark> will verify that we are listening on port 80

### DNS

* The DNS listener allows beacon to send and receive C2 messages over different lookup/response types uncluding A, AAAA, and TXT
* TXT is used by default as it can hold the most data
* This requires us to create one or more DNS records for a domain that the team server will be authoritative for
* Select Beacon DNS

<figure><img src="../.gitbook/assets/image (2) (1) (3).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:red;">Opsec Warning</mark>

* If you run dig on the dns records, the server's <mark style="color:yellow;">default response is 0.0.0.0</mark>
* This is bad because it is defaulted and Cobalt Strike servers can be fingerprinted this way
* This can be changed in the malleable C2 profile

<figure><img src="../.gitbook/assets/image (5) (4) (2).png" alt=""><figcaption></figcaption></figure>

### Peer-to-Peer (P2P)

* Differs from egress listeners because they don't communicate with the team server directly
* Instead, P2P listeners are designed to chain multiple beacons together in parent/child relationships
*
