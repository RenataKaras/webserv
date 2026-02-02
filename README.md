# webserv

## OSI - Open System Interconnection model
- characterizes and standardizes communication functions of a telecommunication or computing system without regard to its underlying internal structure and technology
- it partitions a communication system into abstraction layers (7 of them)
  - to implement HTPP, we only care about 4th layer: Transport layer

## Transport Layer
- primarily responsible for ensuring that data is transferred from one point to another reliably and without errors (for example it makes sure that data is sent and received in the correct sequence)
- provides flow control and error handling
- participates in solving problems concerned with the transmission and reception of packets
- Common examples:
  - **TCP Transmission Control Protocol** -  we use this one to implement the server
    - all the famous HTTP servers are implemented on top of TCP, so that's why we stick with that one
  - UDP User Datagram Protocol
  - SPX Sequenced Packet Exchange

## RFC - A Request for Comments
- publication of technical development and standards-setting bodies for the Internet
- official documents of Internet specifications, communications protocols, procedures and events
- to implement the HTTP server, these are relevant RFCs:
  - [RFC 7230](http://www.rfc-editor.org/info/rfc7230), [RFC 7231](http://www.rfc-editor.org/info/rfc7231), [RFC 7232](http://www.rfc-editor.org/info/rfc7232), [RFC 7233](http://www.rfc-editor.org/info/rfc7233), [RFC 7234](http://www.rfc-editor.org/info/rfc7234), [RFC 7235](http://www.rfc-editor.org/info/rfc7235)

SOCKET
• to implement TCP, we have to learn TCP socket programming
• SOCKET is the mechanism that most operating systems provide to give programs access to the network
∘ allows messages to be sent and received between applications (unrelated processes) on different networked machines
∘ it was created to be independent of any specific type of network, however
∘ IP is the most dominant network and the most popular use of sockets

∘ TCP/IP sockets steps:
‣ Create the socket
• int server_fd = socket
‣ Identify the socket
‣ On the server, wait for an incoming connection
‣ Send and receive messages
‣ Close the socket

