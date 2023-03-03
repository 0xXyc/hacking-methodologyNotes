---
description: Sockets and socket API's!
---

# Socket Code

## Sockets

This technology provides a way to send messages across a network.

This works via sockets and the socket API.

The most common type of socket applications are client-server applications, where one side acts as the server and the other waits for connections from clients.

## Socket API Overview

<mark style="color:yellow;">The primary socket API functions and methods in this module are</mark>:&#x20;

* socket()
* .bind()
* .listen()
* .accept()
* .connect()
* .connect\_ex()
* .send()
* .recv()
* .close()

## TCP Sockets

<mark style="color:yellow;">You can create a socket object using socket.socket(), specifying the socket type as socket.SOCK\_STREAM</mark>. When you do that, the default protocol that's used is the <mark style="color:yellow;">TCP Protocol</mark>.&#x20;

## UDP Sockets

UDP Sockets can be created with socket.SOCK\_DGRAM are NOT reliable, and data read by the receiver can be out-of-order from the sender's writes.

<figure><img src="../../.gitbook/assets/sockets-tcp-flow.webp" alt=""><figcaption></figcaption></figure>

## API Calls that the server makes up a "listening" socket:

* socket()
* .bind()
* .listen()
* .accept()

### How does it work?

A listening socket does exactly what its name suggests. It listens for connections from clients. When a client connects, the server calls .accept() to accept or complete the connection.

The client calls .connect() to establish a connection to the server and initiate the three-way handshake. The handshake step is important because it ensures that each side of the connection is reachable in the network.

<mark style="color:yellow;">In the middle is the round-trip section</mark>. This is where data is exchanged between the client and server using calls to .send() and .recv().

At the bottom, the client and server close their respective sockets.
