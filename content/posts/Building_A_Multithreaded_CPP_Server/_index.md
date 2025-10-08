+++
date = '2025-10-07T19:02:21-04:00'
draft = false
title = 'Building a Multithreaded TCP Server in C++'
+++

Building low-level software has always been an interest of mine, so over the weekend, I decided I wanted to practice some C++, and learn more about TCP/IP. I decided my next C++ challenge would be tackling TCP/IP networking from scratch. The goal: build a multithreaded TCP server without any imported libraries.

Now, I've worked briefly with C++. I've read some of *The C++ Programming Guide* and solved some Project Euler questions in the language. I also have a bit more experience with C, reading the *K&R*, and building and/or reimplementing a handful of small command line tools, as well as a very rudimentary terminal-based text editor. So, I figured this wouldn't be a bad way to learn some low-level networking concepts, such as sockets and learn to implement concurrent solutions.

I also self imposed using no AI-generated code at all while writing this program, that meant no Copilot auto-complete as well.

## First Things First: A Basic, Single-Threaded Server

So, where do we get started? Well, when communicating with two servers, we are going to need to establish a connection between the two computers. Fortunately, using C++ and the OS's powerful system calls, we have just the tools we'll need in the form of **sockets**. Socket's are endpoints that create network connections between different processes and/or computers. In our case, we'll be communicating between a server program, and 1-2 client programs.

A very rudimentary server example written in C++:
```C++
#include <iostream>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

#define PORT 8080

int main() {
	std::cout << "Server is now running. Press 'CTRL+C' to stop" << "\n";
	
	// Initialize our serverSocket
	int serverSocket = socket(AF_INET, SOCK_STREAM, 0);
	
	// Defining server/socket address information
	sockaddr_in serverAddress;
	serverAddress.sin_family = AF_INET;
	serverAddress.sin_port = htons(8080);
	serverAddress.sin_addr.s_addr = INADDR_ANY;
	
	// Bind the socket
	if (bind(serverSocket, (struct sockaddr *)&serverAddress, sizeof(serverAddress)) < 0) {
        std::cout << "Socket binding failed. Exiting." << std::endl;
        return 0;
    } else {
        std::cout << "Socket created and bound." << std::endl;
    }
    // Listen on the socket for a connection
    listen(serverSocket, 5);
    
    while(true) {
        int clientSocket = accept(serverSocket, nullptr, nullptr);
	    char buffer[1024] = { 0 };
	    recv(clientSocket, buffer, sizeof(buffer), 0);
	    std::cout << "Message: " << buffer << "\n";
	}
    
    close(serverSocket);
	return 0;
}
```

And there, we have it, we've built a very basic TCP server! We can connect to it in a variety of different ways and ping it with messages! But first lets breakdown a bit of what we just did. 

Looking at line 12, we can see:
`int serverSocket = socket(AF_INET, SOCK_STREAM, 0);`
Breaking down a few of the keywords here, `AF_INET` stands for Internet Protocol (IPv4), while `SOCK_STREAM` specifies the TCP protocol. This line creates a socket for the server to take input.

### The TCP Three-Way Handshake: Bind, Listen, and Accept

Each TCP server must establish a reliable connection with a client, all the while waiting for *anyone* to call. To do this, we use the three-step handshake known as **Bind-Listen-Accept**.

1. **Bind**: Establishing an Address on the Server
   `bind(serverSocket, (struct sockaddr *)&serverAddress, sizeof(serverAddress);`
   Before our server application can receive any network traffic, we must first let the machine know we are here. Using the `bind()` function, we assign a network **address** and **port number** to the server's socket. The piece of code above tells the computer to route any packets from a specific address and port to the socket in our process.
2. **Listen**: Waiting for a Call
   `listen(serverSocket, 5);`
   Once our socket has an address, we must patiently wait for any connections to come our way. This can be done using the `listen()` function. The line above tells the program to listen on the serverSocket for any connections. It also creates a backlog of 5 requests if the server is not ready to handle them.
3. **Accept**: Answering the Call
   `int clientSocket = accept(serverSocket, nullptr, nullptr);`
   Finally, the `accept()` function completes the connection with the client. The line above creates and stores a new socket to handle the communication with only that client.

## Problems Begin to Show

There are a few problems with this implementation however. First of all, it can't really handle multiple connections at a time. This is because the `accept()` and `recv()` calls are **blocking**--meaning the server will halt and wait on one user to send a message before it is able to process any other user's messages. This is obviously very inefficient for our server program.

## The Solution: Multithreading

To solve this, we'll incorporate some multithreading, creating a thread for each user. Now, this is my first time incorporating threads into a C/C++-style program. Normally, one would use `std::thread`, given it is a portable solution thats apart of the standard library, and to be honest I should've probably used it. Instead, I opted to use the POSIX Threads library, also known as `pthreads`. The primary reason for this was because I found a wonderful video created by Jacob Sorber, discussing how to write a multithreaded server in C. This taught me most of what I implemented.

[Link to the Video](https://www.youtube.com/watch?v=Pg_4Jz8ZIH4)

After studying his approach, and incorporating some more tidbits, I came up with some thing like this:

```C++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <pthread.h>

#define PORT 8080

void* handleConnection(void* clientP) {
	int* client = static_cast<int*>(clientP);
	delete static_cast<int*>(clientP);
	
	while(true) {
		std::string buffer;
		buffer.resize(1024);
		ssize_t byte_count = recv(*client, &buffer[0], buffer.size(), 0);
		if (byte_count <= 0) {
			std::cout << "recv() error: " << strerror(errno) << " " << errno << "\n";
			return nullptr;
		}
		buffer.resize(byte_count);
		std::cout << "Message: " << buffer << "\n";
	}
	
	return nullptr;
}

int main() {
	std::cout << "Server is now running. Press 'CTRL-C' to stop.\n";
	
	// Init serverSocket
	int serverSocket = socket(AF_INET, SOCK_STREAM, 0);
	
	// Init sockaddr_in structure for the Socket Information
    sockaddr_in serverAddress;
    serverAddress.sin_family = AF_INET;
    serverAddress.sin_port = htons(PORT);
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    
	// bind the socket
    if (bind(serverSocket, (struct sockaddr *)&serverAddress, sizeof(serverAddress)) < 0) {
        std::cout << "Socket binding failed. Exiting." << std::endl;
        return 0;
    } else {
        std::cout << "Socket created and bound." << std::endl;
    }
    
	// listening on the selected socket
    listen(serverSocket, 5);
    
	std::cout << "Waiting for connections...\n";
	while(true) {
		socklen_t sockSize = sizeof(socklen_t);
		sockaddr_in clientAddress;
		int clientConnection = accept(serverSocket, (sockaddr*)&clientAddress, (socklen_t*)&sockSize);
		if (clientConnection < 0) {
			std::cout << "Accept failed\n";
			return -1;
		}
		pthread_t thread;
		int *clientP = new int;
		*clientP = clientConnection;
		int threadClient = pthread_create(&thread, NULL, handleConnection, clientP);
	}  
	
	close(serverSocket);
	return 0;
}
```

The server is now multithreaded and able to handle multiple communications at once! The program might be a tad bit longer, but truthfully not much changed. Let's break it down:

Following our `accept()` function, for every successful client connection accepted, a new thread is spawned with the `handleConnection()` function:
`int threadClient = pthread_create(&thread, NULL, handleConnection, clientP);`

We first initialize a pthread type variable named `thread`.
`pthread_t thread;`
We next create a pointer to our client socket.
`int *clientP = new int;`
`*clientP = clientConnection;`
Finally, we call `pthread_create` to start a new thread in the `thread` variable, using the `handleConnection()` function and passing it the `clientP` pointer.

This setup effectively creates a thread for every user connected to the server, allowing each user to send messages to the server at the same time.

(Another important tidbit related to memory management: looking at `handleConnection()`, you can see we free the client pointer we passed once the thread has safely received the socket number. This is to prevent a **memory leak**.)

It was at this point that my goals with this project started to shift. I had completed the goal of building a simple TCP server and even multithreading it to handle multiple clients at once. 

## Next Steps

In just a couple of hours, I had gone from a simple, single-threaded TCP server, to a stronger, multithreaded architecture, and what a journey! I had learned not only some of the nitty-gritty details of network communications, but also how to build a multithreaded solution in C++ using the `pthreads` API. Next up: building a more fully-fledged TCP Chat Client and Server, but thats for another blog post.

