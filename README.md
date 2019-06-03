# event-driven-node

Event-Driven Programming with NodeJS Net and Events.
Go to the profile of Danstan Onyango
Danstan Onyango
Oct 8, 2018

By definition, NodeJS is an event-driven nonblocking runtime environment for JavaScript that has become very popular on the server side. This is because Nodejs has an event-driven architecture capable of asynchronous I/O. This post explores what this event-driven architecture is like and how to exploit the event-driven nature of Nodejs in a very simple and basic way.

Who is this post for?
If you already have been programming in Nodejs, this post is for you because with the available libraries and frameworks in the npmjs community, most developers never get the necessity to experience the core driving capabilities of Nodejs. If you are new to Nodejs, this beginners guide is good for you. As a new beginner, you can also go through this tutorial though you might find some hard time you will start the right way. Either way, you will learn how awesome Nodejs is when approached from this direction.

Case Study.
Before we start event-driven programming, let's find a problem that requires us to use events. I have just the perfect idea! Imagine you have two interfaces. An HTTP client eg a web browser and a TCP interface eg a very monolithic postilion that only accept stream based TCP messages.


By protocol, two things are clear that make the above setup impossible:

The HTTP client only knows how to send an HTTP request and receive an HTTP response while the TCP interface only knows how to receive a message written on a socket and respond to it with another message in a stream-based mode.
The HTTP client can only communicate with an HTTP web server while the TCP interface can only communicate with a TCP client via a socket.
Using events to solve the Problem.
To solve this problem, we have to introduce two entities. A web server and a TCP Client to act as middlemen.


So with the new design, we have the HTTP client send requests to the web server which uses the TCP client to write the message to the socket connecting to the TCP server which on getting the stream message writes back to the socket. You will notice that at the TCP socket layer, everything is event based. Now the web server request handler must use events to wait for the message at the client which also has to use an event to notify the web server when the TCP server writes back a response message.

Prerequisites.
Now that we have a problem to solve and we know how to solve it, let's start writing some cool event listeners and event emitters. Before we begin, here are a few things to revisit.

ES6: This post uses some of the latest features of JS described in this highlights of top ES6 features.
Nodejs Net: Since our problem uses raw TCP, we need the Nodejs net module. This module is also fully event based which makes using it a good learning experience for event-driven programming.
Expressjs: We use express as the web server although we can create one for ourselves like this one here.
Nodejs Events: This module will interface TCP and HTTP.
A brief big picture of Nodejs events

Node is single threaded and achieves nonblocking architecture through an event loop. So each function is executed upon emission of an event. This is the basic idea we want to see here.

Alright, now that we have everything we need, let jump right in.

TCP Client and TCP server
Let's create the TCP client and server that we will connect the HTTP client to.

As for the TCP server, this is what we shall have. This server listens on a port that we can connect to and get information about HTTP status codes. You will notice that all the code here is evet based like when a new connection is created, the connection event is emitted and the listener is called with the socket associated with the connection as the argument. The same applies to the subscribers like on data, error etc that are created to listen on the socket


Now we have the TCP server, let's test it. First, run the server. If you are on windows, just fire up putty and select telnet then enter 127.0.0.1 in hostname and 9670 as the port. as shown below. Click open, you should have a new terminal like a window. Typing 404 or 500 should make a conversation over the socket created.


If you are on Unix, telnet should come by default. If not google up what package has telnet for your OS. You should end up being able to issue

telnet 127.0.0.1 9670
This should open a connection that is writable like this.


How about a TCP client in Nodejs. We need a TCP client that will be used by the web server to communicate with the TCP server. The client will look like this.


You should notice that we do not have an on data event listener in this client. This is on purpose because the data listener will be created at the web server file. This is why we have exported the client. When you run the client on its own, it should connect to the server just fine.


Creating the web server.
Install express and create a server just one endpoint. We call the server with query statusCode being the code we want to get its description from the TCP server. The web server will take the code in the query and write to the socket. Then await a response message. We will use Nodejs event module to make this work. Here is the complete code of the web server.


Now as you can see we use Nodejs event module to create an event listener after writing to the socket. The event listener awaits an event whose name is the code sent in the status query. Upon the TCP client getting the response via its data event listener we emit an event whose name is the status code we received from the HTTP request. Here is a snippet of a sample request to this server that will work when the TCP server is running.

curl localhost:8090?statusCode=500

Now we have a complete working event-based application. However, event-based programming comes with a lot of responsibilities.

Key things to keep in mind.
Memory leaks: Usually JavaScript has almost no memory management. But when you start to use Nodesj core modules like net, events, and streams, some level of memory management is very handy.
Simplicity: With this kind of listeners and emitters all over, it's very easy to get lost in your own code however simple it may be. Here being simple, modular and obstructive helps the most.
Do not use anonymous functions with listeners. In my experience, you will always want to terminate and event listeners/handlers in memory, but the becomes hard when the listener is an anonymous function. Use named functions instead.
Debug memory usage of the application before deployment. This will help you with a whole lot of trouble in the future.
Beyond this article.
You may have liked this post and you're asking what next after knowing how to execute code that is fully event based. Well, here are a few things that I am also still looking at.

Running event-based applications in a cluster such that the exchange of data is done through IPC or an in-memory database like Redis. This is ideal for microservices.
Writing memory-efficient applications in low-level C++ and executing in Nodejs. This makes memory management a walk in the park.
Using typescript for code that requires type checking especially with buffers and socket streams. This will help you with IoT applications that are deployed using Node.
