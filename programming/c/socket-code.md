---
description: 05/31/2022
---

# Socket Code

## Introduction

**Socket programming** is a <mark style="color:yellow;">way of connecting two or more nodes on a network in order to communicate with one another</mark>. This can be used for both legitimate and malcious purposes.&#x20;

This system follows what is known as a <mark style="color:yellow;">client/server</mark> model.

## What are Sockets?

Sockets are low level endpoints used for processing information across a network.

## How does it work?

### Server Process

1. **Socket Creation**

```
int sockfd = socket(domain, type, protocol)
```

* `sockfd`: socket descriptor, an integer
* domain: integer, specifies communication domain. We can use `AF_LOCAL` as defined in the POSIX standard for communication between processes on the same host. <mark style="color:yellow;">For communicating on different hosts connected by IPv4, you will want to use</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`AF_INET`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">and</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`AF_INET 6`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">for processes connected via IPv6</mark>
* type: communication type
* protocol: Protocol value for IP, which is 0
  * This will appear as a number on protocol field in the IP header of a packet (run man protocols for more info)

2. **Setsockopt**

This helps in manipulating options for the socket referred by the file descriptor sockfd. This is completely optional.

However, <mark style="color:yellow;">it helps with reuse of address and port. It prevents the error such as: "Address already in use"</mark>.

```
int setsockopt(int sockfd, int level, int optname,  const void *optval, socklen_t optlen);
```

<mark style="color:yellow;">NOTE</mark>: `setsockopt` is only found in server socket code.

3. **Bind**

```
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

After the creation of the socket, the bind function will then _bind_ the socket to the address and port number specified in addr (custom data structure).

4. **Listen**

```
int listen(int sockfd, int backlog);
```

This will put the server socket in a _passive_ mode, where it will wait for the client to make a connection to the server.

Here exists what is known as the <mark style="color:yellow;">backlog</mark>, <mark style="color:yellow;">which defines the maximum length of a queue for pending connection for sockfd to grow</mark>.

If a client request arrives and the queue is full, the client may receive an error that indicates ECONNREFUSED.

5. **Accept**

```
int new_socket= accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

It extracts the first connection request on the queue of pending connections for the listening socket, `sockfd`, creates a new connected socket, and returns a new file descriptor for that socket.

<mark style="color:yellow;">This is where the connection is granted life and has been established</mark>.

<mark style="color:green;">A connection has now been established between client and server and are therefore ready to transfer data</mark>.&#x20;

### Client Process

1. **Socket Connection**

<mark style="color:yellow;">This works exactly the same as</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">**Socket Creation**</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">for the</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">**Server Process**</mark>.

Please refer to step 1 above for more information/clarity.

2. **Connect**

The <mark style="color:yellow;">**connect() system call**</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">connects the socket referred to by the file descriptor</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`sockfd`</mark> to the address specified by <mark style="color:yellow;">`addr`</mark>. Server's address and port is specified in `addr`.

```
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

## Code Examples

### Server.c

```c
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 1337
int main(int argc, char const* argv[])
{
	int server_fd, new_socket, valread;
	struct sockaddr_in address;
	int opt = 1;
	int addrlen = sizeof(address);
	char buffer[1024] = { 0 };
	char* hello = "Hello from server";

	// Creating socket file descriptor
	if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		perror("socket failed");
		exit(EXIT_FAILURE);
	}

	// Forcefully attaching socket to the port 8080
	if (setsockopt(server_fd, SOL_SOCKET,
				SO_REUSEADDR | SO_REUSEPORT, &opt,
				sizeof(opt))) {
		perror("setsockopt");
		exit(EXIT_FAILURE);
	}
	address.sin_family = AF_INET;
	address.sin_addr.s_addr = INADDR_ANY;
	address.sin_port = htons(PORT);

	// Forcefully attaching socket to the port 8080
	if (bind(server_fd, (struct sockaddr*)&address,
			sizeof(address))
		< 0) {
		perror("bind failed");
		exit(EXIT_FAILURE);
	}
	if (listen(server_fd, 3) < 0) {
		perror("listen");
		exit(EXIT_FAILURE);
	}
	if ((new_socket
		= accept(server_fd, (struct sockaddr*)&address,
				(socklen_t*)&addrlen))
		< 0) {
		perror("accept");
		exit(EXIT_FAILURE);
	}
	valread = read(new_socket, buffer, 1024);
	printf("%s\n", buffer);
	send(new_socket, hello, strlen(hello), 0);
	printf("Hello message sent\n");

	// closing the connected socket
	close(new_socket);
	// closing the listening socket
	shutdown(server_fd, SHUT_RDWR);
	return 0;
}
```

### Client.c

```c
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 1337

int main(int argc, char const* argv[])
{
	int status, valread, client_fd;
	struct sockaddr_in serv_addr;
	char* hello = "Hello from client";
	char buffer[1024] = { 0 };
	if ((client_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		printf("\n Socket creation error \n");
		return -1;
	}

	serv_addr.sin_family = AF_INET;
	serv_addr.sin_port = htons(PORT);

	// Convert IPv4 and IPv6 addresses from text to binary
	// form
	if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)
		<= 0) {
		printf(
			"\nInvalid address/ Address not supported \n");
		return -1;
	}

	if ((status
		= connect(client_fd, (struct sockaddr*)&serv_addr,
				sizeof(serv_addr)))
		< 0) {
		printf("\nConnection Failed \n");
		return -1;
	}
	send(client_fd, hello, strlen(hello), 0);
	printf("Hello message sent\n");
	valread = read(client_fd, buffer, 1024);
	printf("%s\n", buffer);

	// closing the connected socket
	close(client_fd);
	return 0;
}
```
