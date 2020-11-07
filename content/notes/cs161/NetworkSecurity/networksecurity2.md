# Network Security: Part 2
### The Network Stack

One of the fundamental models in computer networks is the **protocol stack**. The protocol stack is a set of services stacked such that each layer depends on the layers of services below it. There are actually seven layers, but in reality there are only 4 important layers, arranged from top to bottom:

1. Application Layer
2. Transport Layer
3. Network Layer
4. Link Layer

You might know this as the **TCP/IP stack**. 

Let's briefly look through each layer:

The **link layer** is responsible for linking data between hosts: i.e. making the communication **in a LAN**.  Some (hopefully) familiar link layer protocols are Wi-Fi and Ethernet. 

The **network layer** is responsible for actually finding (good) routes in the network for a specified connection, handling stuff like routing and network congestion. The **Internet Protocol** (IP) is an *extremely* important network layer protocol. 

The **transport layer** handles the completed, fuckup-free sending of traffic between hosts, even in a WAN. Some important transport layer protocols are **Transmission Control Protocol** (TCP), which deals with order of delivery of messages and error recovery, and **User Datagram Protocol** (UDP), which is like a faster and less expensive TCP but not as secure. 

Finally, the **application layer** handles applications communicating with one another, like Web browsers talking to web servers through **Hypertext Transfer Protocol** (HTTP).

Data flows down the sender's stack, gets sent to the receiver, and then goes up the receiver's stack.

```html
<!---
Insert a diagram of data flow across the protocol (network) stack.
-->
```

A lot of abstraction here. It'll be a lot easier with an example. Let's use our familiar case: we want to visit the GOATed site www.cs61a.org. In this scenario, we have our browser host machine trying to communicate with a server associated with www.cs61a.org. Starting at the application layer, the browser sends an HTTP request to the server. Then, our browser utilizes the transport layer to establish a secure connection to the server. The transport layer appends a header to the request and sends the message to the network layer. The network layer then fragments the message into packets and sends it to the link layer, which places the packets into an Ethernet/WiFi frame and sends it over to the destination host. Then, once **all** packets are received by the destination host, the packets are re-assembled into the original request (note this is just the REVERSE of the process above!) and delivered to the cs61a server at the application layer. The server has now successfully received the http request the browser sent! 