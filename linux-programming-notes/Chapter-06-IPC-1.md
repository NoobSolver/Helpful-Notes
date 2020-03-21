# Inter-Process Communication (IPC)

Though you can do workarounds to do inter-process communication like:

* using file locks to synchronize processes
* using signals to send messages

You get the real power of IPC from pipes, FIFOs, sockets and so on, which are natively integrated to the kernel.

There are generally two categories of IPC mechanisms on Linux:

1. Files: Pipe, FIFO, Socket
1. System V IPC: semaphore, shared memory, message queue

The IPC mechanisms in the first category provides an interface of file descriptor and is more consistent with the UNIX philosophy.

System V IPC mechanisms tend to be more complicated (the implementation of semaphore is rather *bad*).

## Pipe

Pipe is most commonly used in shell. As long as you use the pipe operator, `|`, you are using pipe to do IPC. For one-way data flow, pipe is fairly adequate and handy to use. As a result of its onw-way nature, pipes are said to be *half-duplex*. Pipes have a limitation that they can be used only when processes with the same ancestor.

In terms of implementation, a pipe is actually a file, but anonymousi (not exist in the file system). The size of pipe is limited to `PIPE_BUF` in kernel.

### Pipe Creation

Use `pipe` to create a pipe:

    #include <unistd.h>
    int pipe(int filedes[2]);

after calling `pipe`:

* `filedes[0]` is opened for reading
* `filedes[1]` is opened for writing

Programmers can use `pipe` to let parent process communicate with another program as its child. The steps are as follows:

1. the parent process creates a pipe
1. the parent process forks a child process
1. in the parent process `filedes[0]` is closed while in child `filedes[1]` is
1. a pipe is formed

It is also common practice to use `dup` to replace child process' file descritpr 0, the `STD_IN` with `filedes[0]`. With this technique, the forming of pipe is transparent to the child process.

If *full duplex*, i.e. two-way communication,  is desired, another pipe should be created.

### `popen`

`popen` is a convenient way to establish a one-way communication channel with child process.

    #include <stdio.h>
    FILE *popen(const char *command, const char *type);
    int pclose(FILE *stream);

* `popen` with "r" would redirect child process' `stdout` to fp returned by `popen`
* `popen` with "w" would redirect child process' `stdin` to fp returned by `popen`

## FIFO

FIFO is named pipe and it resides in the file system. As its name indicates, a FIFO has a _First In First Out_ behavior and can have multiple processing read and write at the same time. So careful consideration is needed to deal with synchronization.

FIFOs can be created with `mkfifo` or `mknod` commands or functions. To delete FIFOs, use `rm` command or `unlink` call.

### Opening a FIFO

As FIFO is a special kind of file, opening it in different modes can result in different effects:

* simply open/read a FIFO will block the process, until another process read/write it
    * if not explicitly declared, FIFOs are opened with `O_BLOCK` mode
* if you `open` a FIFO with `O_WRONLY` and `O_NONBLOCK`, and if nobody is reading, -1 will be returned
* if you `open` a FIFO with `O_RDONLY` and `O_NONBLOCK`, the fd is returned immediately

As far as writing is concerned, the behavior is similar:

* in `O_BLOCK` mode, read or write will block until someone writes or reads
* in `O_NONBLOCK` mode, read or write with no writer or reader will return -1

### Trick with FIFO

We can use FIFO along with `tee` command to send output to two programs simutaneously.

    command tee: tee [file]
    output to file and stdout simultaneously

Suppose we've got a FIFO named `my_fifo`, the following commands send output of `prog1` to `prog2` and `prog3`:

    $ prog2 < my_fifo &
    $ prog1 | tee my_fifo | prog3

We can also use FIFO to implement C/S architecture:

                               +--------+
                      <<<<<<<<<| server |>>>>>>>>>
                     /         +--------+         \
                    /              ^               \
           response/               ^ request        \ response
                  /                ^                 \
    +---------------+       +--------------+       +---------------+
    | RESPONSE FIFO |       | REQUEST FIFO |       | RESPONSE FIFO |
    +---------------+       +--------------+       +---------------+
                  \         /              \         /
           response\       /request  request\       / response
                    \     /                  \     /
                  +---------+              +---------+
                  | client1 |              | client2 |
                  +---------+              +---------+

## System V IPC

UNIX System V introduced three new ways of IPC communication:

* Semaphore set
* Message queue
* Shared memory

System V IPC introduced the concepts of identifier and keyword:

* IPC object are referenced by identifier
* create IPC object by specifying a key of type `key_t`
    * pre-defined constant: `IPC_PRIVATE`
    * `ftok` function
* the kernel convert key to identifier

the signature of `ftok` is as follows:

    #include <sys/types.h>
    #include <sys/ipc.h>
    key_t ftok(const char *):

There is also a `struct ipc_term` for permission controlling.

**Criticism**: by introducing new concepts, System V IPC breaks the UNIX philosophy of _everything is file_.

### SV IPC System Calls overview

feature          |    message queue     |  semaphore        | shared memory
-----------------|----------------------|-------------------|----------------------
allocate an IPC object |      `msgget`          | `semget`            | `shmget`
send/receive message, etc     |      `msgsnd`/`msgrcv`          | `semop`             | `shmat`/`shmdt`
IPC control      |      `msgctl`          | `semctl`            | `shmctl`

**Criticism**: in the POSIX standard interface, semaphores are created/manipulated as sets. While the intention might be good, in reality semaphores are manipulated one at a time. This design just make the delicate IPC more complicated.
