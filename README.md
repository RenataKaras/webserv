# webserv

## Table of Contents
- [OSI - Open System Interconnection model](#osi---open-system-interconnection-model)
- [Transport Layer](#transport-layer)
- [RFC - A Request for Comments](#rfc---a-request-for-comments)
- [Socket](#socket)
  - [Create the socket](#create-the-socket)
  - [Identify (name) the socket](#identify-name-the-socket)
  - [On the server, wait for an incoming connection](#on-the-server-wait-for-an-incoming-connection)
  - [Send and receive messages](#send-and-receive-messages)
  - [Close the socket](#close-the-socket)
- [HTTP (Hypertext Transfer Protocol)](#http-hypertext-transfer-protocol)
  - [HTTP Client](#http-client)
  - [HTTP Request](#http-request)
  - [HTTP Methods](#http-methods)
  - [HTTP Server / Response](#http-server--response)
- [Non-Blocking Sockets](#non-blocking-sockets)
  - [Blocking Sockets](#blocking-sockets)
  - [Non-Blocking Sockets](#non-blocking-sockets-1)
  - [I/O Multiplexing and epoll](#io-multiplexing-and-object-object)


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
    ```c
    int server_fd = socket(domain, type, protocol)
    ```
    - **domain**, or address family is a communication domain in which the socket should be created
      - **where?** (IPv4)
      - `AF_INET (IP)`, `AF_INET (IPv6)`, `AF_UNIX (local channel, similar to pipes)`, `AF_ISO (ISO protocols)`, `AF_NS (Xerox Network Systems protocols)`
    - **type** of service, selected according to the properties required by the application
      - **how?** (stream vs datagram)
      - `SOCK_STREAM (virtual circuit service)`, `SOCK_DGRAM (datagram service)`, `SOCK_RAW (direct IP service)`
    - **protocol**, indicate a specific protocol to use in supporting the sockets operation
      - **exact rulebook**, we use 0 because AF_INET + SOCK_STREAM combo = TCP
      - useful in cases where some families may have more than one protocol to support a given type of service
      - 0 for a protocol means *choose the **default protocol** for this address family + socket type*
    - return value is a file descriptor (small int), domain is `AF_INET` because we want to specify the IP address family, and type is `SOCK_STREAM`, protocol is 0

  - ### Identify (name) the socket
    - assigning a transport address to the socket (a port number in IP networking)
    - Which protocol family is this address for? (IPv4, IPv6, ...), What concrete address values are we binding? (IP, port)
    - this operation is called ***binding an address*** and the **bind** system call is used for this
      ```c
      int bind(int socket, const struct sockaddr *address, socklen_t address_len);
      ```
      - `socket` is the socket that was created with the *socket* system call (a fd), structure `sockaddr` is a generic wrapper that allows the OS to be able to read the first couple of bytes that identify the address family, and `address_len` says how many bytes defines the address
      - `*address` is a pointer to a real address struct based on the generic interface of `sockaddr`, for IP networking, we use `struct sockaddr_in`, which is predefined in `netinet\in.h`:
   
        ```c
        struct sockaddr_in {
            __uint8_t      sin_len;
            sa_family_t    sin_family;
            in_port_t      sin_port;
            struct in_addr sin_addr;
            char           sin_zero[8];
        };
        ```


        - before calling *bind*, we need to fill out this structure with three key parts:
          1. `sin_family` -  the address family we used when we set up the socket (`AF_INET`)
          2. `sin_port` - the port number (the transport address)
              - **client** that doesn't receive any incoming connections just lets the operating system pick any available port number by specifying port 0
              - **server** picks a specific number since clients will need to know a port number to connect to
          3. `sin_addr` - the address for this socket, which is your machine's IP address
              - most of the time we don't care to specify a specific interface and can let the OS use whatever it wants, the special address for this is 0.0.0.0., or `INADDR_ANY`
           
  - ### On the server, wait for an incoming connection
    - before a client can connect to a server, the server should have a socket that is prepared to accept the connection
    - `listen` system call tells a socket that it should be capable of accepting incoming connections
        ```c
        int listen(int socket, int backlog);
        ```
      - `backlog` defines the maximum number of pending connections that can be queued up before connections are refused
    - `accept` system call grabs the first connection request on the queue of pending conncetions (that we set up in `listen`) and creates a **new socket** for that connection
      - the original socket that we made for listening is used *only* for accepting connections, but not for exchanging data
      - by default socket operations are synchronous or **blocking**, and accept will block until a connection is present on the queue

        ```c
        int accept(int socket, struct sockaddr *restrict address, socklen_t *restrict address_len);
        ```
        - `socket` is the socket we originally set up for accepting connections with `listen`
        - `address` is the address struct that will be, after we call `accept`, filled out with client's address(IP and port), and the third parameter is filled in with the length of the address structure
  - ### Send and receive messages
    - communication should be the easy part, the same `read` and `write` that work on files, work on sockets:

      ```c
      char buffer{1024} = {0};
      int valread = read(new_socket, buffer, 1024);
      printf("%s\n", buffer);
      if (valread < 0)
        printf("There are no bytes to read from client\n");

      char *hello = "Hello from the server\n";
      write(new_socket, hello, strlen(hello));
      ```
  - ### Close the socket
    - ```c
      close(new_socket);
      ```

## HTTP (Hypertext Transfer Protocol)
- outline of the interaction between Web Browser and HTTP Server:
  1. HTTP client (for example, web browser) sends a HTTP request to the HTTP server
  2. Server processes the request received and sends HTTP response to the HTTP client
 
- ### HTTP Client
  - Client needs to connect to the server every time, while server **can't** connect to the client
  - it's the duty of the client to initiate the connection - when we want to connect to the server we type in a URL/Address of the website in the browser `http://www.example.com:80/index.html`
    - to display the page, browser fetches the file `index.html` from a web server

- ### HTTP Request
  1. Header: 
      ```http
      GET /index.html HTTP/1.1
      Host: www.example.com
      User-Agent: Mozilla/5.0
      Accept: text/html, */*
      Accept-Language: en-us
      Accept-Charset: ISO-8859-1,utf-8
      Connection: keep-alive
      ```
    
  - `GET` is a method, `/index.html` is a URL, and `HTTP/1.1` is a protocol version
  2. Then blank line that marks the break between the header and the body
  3. Body (optional, for POST and PUT method)

- ### HTTP Methods
  - There are 9 all together, and some of those 9 are:
    1. GET - Fetch a URL
    2. HEAD - Fetch information about a URL
    3. PUT - Store to an URL
    4. POST - Send form data to a URL and get a response back
    5. DELETE - Delete a URL
       - **GET** and **POST** are most commonly used. Other than these two, we have to implement **DELETE** as well as a part of this project.

- ### HTTP Server / Response
  - the browser is expecting same format response in which it sent us the request:
    1. Header:
        ```http
        HTTP/1.1 200 OK
        Date: Fri, 16 Mar 2018 17:36:27 GMT
        Server: *name_of_my_server*
        Content-Type: text/html;charset=UTF-8
        Content-Length: 1846
        ```
    - `HTTP/1.1` is a version, `200` is status and `OK` is a status message
    2. Then we have a blank line that marks the break between the header and the body
    3. Body:
        ```http
        <?xml ... >
        <!DOCTYPE HTML ... >
        <html ... >
        ...
        </html>
        ```
  - **Status code and Status Messages**
    - issued by a server in response to a client's request made to the server
    - first digit of the status code specifies one of five standard classes of responses:
      1. 1xx - informational response
      2. 2xx - success
      3. 3xx - redirection
      4. 4xx - client error
      5. 5xx - server error 
    - the message phrases shown are typical, but any human-readable alternative may be provided
    - for example if I can't find the file that client is asking, or if the client has no permission to see the file, I send an appropriate code that corresponds with that
    - [List of the status codes that can be used](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

   
  - **EXAMPLE** - we get `GET /info.html HTTP/1.1`
    - we have to search for the `info.html` file in the **root directory of the server**
      -  options are: the file is present, the file is absent, the client doesn't have permission to access the file etc.
      -  we select the appropriate status code
      -  if the file is present, and the client has permission to access it, then we select the appropriate `Content-Type`
      -  then we open the file and read the data into a variable. Count the number of bytes read from the file. We set the `Content-Length` with that value.
      -  then we construct the **Response Header**
      -  add a `newline` at the end of the **Response Header**, and then we append data we have read from the file to it
      -  send the response to the client!
      -  DONE

## Non-Blocking Sockets
- ### Blocking Sockets
  - by *default*, socket operation in C are **blocking**
    - when we call a function like `accept()`, `read()` or `write()`, the program will halt and wait until that operation is complete
    - `accept` blocks until a new client connects, `read` blocks until there is data to be read from the client, and `write` blocks until the data has been sent to the kernel's buffer
  - **problem** is that a server with a blocking socket can only handle one client at a time, and cannot accept new connections or service other clients
  - Traditional solutions:
    1. Multi-threading: dedicate a thread to each client (resourse intensive and complex to manage)
    2. Multi-processing (`fork()`): create a new process for each client (even more resource-intensive than threading)

- ### Non-Blocking Sockets
  - non-blocking socket will never halt a program, if an operation cannot be completed immediately, it will return an error
    - `accept()` if no clients are waiting, it returns an error
    - `read()` if there's no data to read, it returns an error
    - `write()` is the kernel's send buffer is full, it returns an error
  -  the socket can be set to non-blocking one with `fcntl()`(file control) function to change the properties of a file descriptor
      ```c
       fcntl(sockfd, F_SETFL, flags | O_NONBLOCK)
      ```
  - **new problem** - if we just set the socket to non-blocking and try to read from it in a loop, the `read()` will return immediately, no data gives `EAGAIN`, loop runs again, calls `read()`, still no data, and this repests millions of times per second which makes CPU run at 100%
    - this is called ***busy-waiting*** - spinning in a loop checking something over and over

 - ### I/O Multiplexing and [object Object]
   - I/O multiplexing allows us to monitor multiple file descriptors (sockets) at the same time and get notified when one of them is ready for an I/O operation (ready to be read from or written to)
   - `epoll` is a modern and efficient I/O multiplexing API on Linux
     - more scalable than `select()` and `poll()` because it's performance doesn't degrade as the number of monitored file descriptors increases

   - **The [object Object] API**
     - 3 main functions of `epoll` API:
       1. ```c
          int epoll_fd = epoll_create1(0);
          ```
          - creates a **kernel object** whose job is to "keep a list of file descriptors and wake us up when any of them is ready", a sort of a **epoll manager**
       2. ```c
          epoll_ctl(epoll_fd, op, target_fd, &event);
          ```
          - adds, modifies, or removes file descriptors from the `epoll` instance's 'interest list'
           - `epoll_fd` the file descriptor for the epoll instance
           - `op` the operation to perform:
             - `EPOLL_CTL_ADD` add `target_fd` to the interest list
             - `EPOLL_CTL_MOD` modify the events for `target_fd`
             - `EPOLL_CTL_DEL` remove `target_fd` from the interest list
          - `target_fd` the file descriptor we want to monitor (e.g. server socket or a client socket)
          - `&event` a pointer to a `struct epoll_event`. This struct tells `epoll` what events are we interested in for `target_fd`

            ```c
            struct epoll_event {
            uint32_t      events;  /* Epoll events */
            epoll_data_t  data;    /* User data variable */
            };
            ```
            - `events` a bitmask for events, common ones:
              - `EPOLLIN` the associated file is available for `read()` operations
              - `EPOLLOUT` the associated file is available for `write()` operations
              - `EPOLLET` sets edge-triggered behaviour
                - **Edge-triggered(ET) vs Level-triggered(LT) events**
                  1. Level-triggered (LT) is the default
                      - `epoll_wait` will continuously report an event as long as the condition holds
                  2. Edge-triggered (ET) - `epoll_wait` will only report an event *once* when the state changes
                      - it's more efficient because it prevents `epoll_wait` from constantly reminding us about an event that hasn't been handled yet
                      - it has a catch -  when we get ET notification, the file descriptor **must** be processsed until it would block - if we only read part of the data, `epoll` won't notify us about it again and it will sit in the buffer forever
                      - using while loops aroung `accept()` and `read()` to avoid this scenario
            - `data` is for us to use, and anything can be stored in it
              - it is common to store the file descriptor itself (`event.data.fd = target_fd;`) or a pointer to a struct containing client state

       3. ```c
          epoll_wait(epoll_fd, events, max_events, timeout);
          ```
          - this is the core of the event loop, it waits for events on the file descriptors in the interest list
          - `epoll_fd` is the `epoll` instance file descriptor
          - `events` is a pointer to an array of `struct epoll_event` that will be filled with information about the events that have occured
          - `max_events` is the maximum number of events `epoll_wait` should return in a single call
          - `timeout` is the maximum time to wait in milliseconds, a value of `-1` means wait indefinitely, a value of `0` means return immediately, even if there are no events
          - **return value** is the number of file descriptors ready for the requested I/O, or `0` if it timed out, or `-1` on error 
