## Sockets

Communication over sockets is a widely used technique, especially in web applications. On Linux, local sockets and Internet sockets share a unified interface, and thus programs can use sockets to do IPC.

Basically, IPC over sockets is implemented in a client-server architecture, as the process which makes request acts as client and the process which sends response acts as server. It is often that a server can serve mutiple clients at the same time.

### Types of Sockets Connection

There are two types of socket connection: connection-oriented and connectionless. 

Connection-oritend sockets ensures reliablity of connections with automatic packet ordering, resending if missing and flow control. Although these impose some burden on performance, the default behavior is desiarable in most settings. In contrast, connectionless sockets does not ensure the reliability of connection and various problems are directly left for programmer to deal with, for example:

* orderless packets
* lost packets
* resend request
* duplicate packets

When using connectionless sockets, the data integrity checking should be manually done by programmers. And albeit the disadvantages of connectionless sockets, they are useful in some specific circumstances, where only one-time data fetch is needed and loss of data may not be harmful. Many application layer protocols are based on connectionless sockets, such as DNS while others provide both connection-oriented and connectionless versions.

In terms of Internet sockets, TCP and UDP are used respectively.

When programmed, the two types of sockets are established with different interfaces:

* connection-oriented: `listen`&`accept`&`connection`
* connectionless: `sendto`&`recvfrom`

### Establishing Socket Connection

Below is the common process for establishing a connection-oriented socket connection.

Process of establishing socket connection at server side:

1. create a socket via `socket`
2. bind socket to specific address via `bind`
3. make socket passive and start listening via `listen`
4. start a loop and `accept` requests

Process of establishing socket connection at client side:

1. create a socket via `socket`
2. `bind` to address (optional)
3. connect to server address via `connect`

### I/O Models

Just like file I/O, there are several I/O models when doing socket communication and each has unique characteristics. If performance is high concern, programmers should pay special attention to I/O model.

#### Block Model

This is the default behavior, and is generally of low performance as every read/write will block until write/read is done at the other side.

#### Non-Block Model

This model is also similar to that of file I/O. With non-blocking read or write, no blocking will happen and performance will rise. The problem of this model is that as read/write can fail, mutiple times of trail of reading or writing can be elusive.

#### I/O multiplexing

This model is made possible by `select` call:

    #include <sys/time.h>
    #include <sys/types.h>
    #include <unistd.h>

    int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

`select` blocks until one of the fd in `fd_set`s is ready. And subsequent read/write on that fd will not block. Putting `select` before `accept` can greatly boost performance.

#### Signal-Driven I/O

This model is done via the signal `SIGIO` and setting some flag on file descriptor using `fcntl`, e.g.:

    fcntl(conn_fd, F_SETOWN, pid);
    
The statement above will cause a signal sent to process with `pid` if something happened on `conn_fd`.

### Host Address and Service Port Query

Host address can be obtained via different machanisms and can be done programmatically.

    #include <netdb.h>
    struct hostent *gethostbyname(const char *name);
    struct hostent *gethostbyaddr(const char *name, size_t len, int type);

Many well-known services' port number can be obtained via functions. Generally, the functions looks up the /etc/services file.

    #include <netdb.h>
    struct servent *getservbyname(const char *name, const char *proto);
    struct servent *getservbyport(int port, const char *proto);
