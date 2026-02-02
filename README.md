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

## Socket
- to implement TCP, we have to learn TCP socket programming
- **SOCKET** is the mechanism that most operating systems provide to give programs access to the network
- allows messages to be sent and received between applications (unrelated processes) on different networked machines
- it was created to be independent of any specific type of network, however
- **IP** is the most dominant network and the most popular use of sockets

- TCP/IP sockets steps:
  - ### Create the socket

    ```int server_fd = socket(domain, type, protocol)```
    - **domain**, or address family is a communication domain in which the socket should be created
      - **where?** (IPv4)
      - ```AF_INET (IP)```, ```AF_INET (IPv6)```, ```AF_UNIX (local channel, similar to pipes)```, ```AF_ISO (ISO protocols)```, ```AF_NS (Xerox Network Systems protocols)```
    - **type** of service, selected according to the properties required by the application
      - **how?** (stream vs datagram)
      - ```SOCK_STREAM (virtual circuit service)```, ```SOCK_DGRAM (datagram service)```, ```SOCK_RAW (direct IP service)```
    - **protocol**, indicate a specific protocol to use in supporting the sockets operation
      - **exact rulebook**, we use 0 because AF_INET + SOCK_STREAM combo = TCP
      - useful in cases where some families may have more than one protocol to support a given type of service
      - 0 for a protocol means *choose the **default protocol** for this address family + socket type*
    - return value is a file descriptor (small int), domain is ```AF_INET``` because we want to specify the IP address family, and type is ```SOCK_STREAM```, protocol is 0

  - ### Identify (name) the socket
    - assigning a transport address to the socket (a port number in IP networking)
    - Which protocol family is this address for? (IPv4, IPv6, ...), What concrete address values are we binding? (IP, port)
    - this operation is called ***binding an address*** and the **bind** system call is used for this

      ```int bind(int socket, const struct sockaddr *address, socklen_t address_len);```
    - ```socket``` is the socket that was created with the *socket* system call (a fd), structure ```sockaddr``` is a generic wrapper that allows the OS to be able to read the first couple of bytes that identify the address family, and ```address_len``` says how many bytes defines the address
    - ```*address``` is a pointer to a real address struct based on the generic interface of ```sockaddr```
    - for IP networking, we use ```struct sockaddr_in```, which is predefined in ```netinet\in.h```:
   
      ```struct sockaddr_in 
        { 
            __uint8_t         sin_len; 
            sa_family_t       sin_family; 
            in_port_t         sin_port; 
            struct in_addr    sin_addr; 
            char              sin_zero[8]; 
        };
  - On the server, wait for an incoming connection
  - Send and receive messages
  - Close the socket

