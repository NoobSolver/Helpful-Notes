# Files

## File in Linux

File is the abstraction of static resource in linux. Generally speaking, the file concept can be applied to resources on storage devices (e.g. a text document) as well as in kernel (e.g. a pipe).

With an emphasis on openness and unity, UNIX organizes files as byte streams with no particular internal structure. *Plain ASCII text is simple yet powerful and an ecosystem of tools is build on that.*

## Virtual File System

With the concept of mounting, UNIX/Linux organizes all files as a single tree representation, which provides a consistent and predictable way for accessing files. The tree structure is said to be virtual as it is only the logical representation and not all file in it actually exist.

To be more specific, the VFS:

* is an abstraction layer of common file system organization
* is like a bus, access to mounted file systems
* is like a switch
* is implemented by the f\_op pointer in file structure to indicate the file system type
* exists in memory, as *virtual* indicates
* manages a buffer in memory, which maps to files on disk

## File Systems

File system is the way how data is organized on storage devices.

Some oldest UNIX file system contains: 

* boot block
* super block
* inode table and data block

In UNIX, file name is not part of a file's attributes (in inode). Instead, file names are stored in directory entries.

### `ext` File System Family

The most commonly used file systems in Linux. `ext` means extension. `ext2` is the first widely used file system of this family. `ext3` provides journaling and `ext4` provides more modern functionalities.

Data space is splitted into blocks, which are grouped. Each block group contains a copy of the superblock, which is crucial to the booting of the OS.

Every file or directory is represented by an inode, which is known as index node. Inodes contain information of files and have a structure of hierarchical block pointers. Take `ext2` inode as example, in each inode there is:

1. pointers to direct blocks
1. 1 pointer to indirect blocks
1. 1 pointer to double indirect blocks

Based on block size, `ext2` can have different maximal file size and filesystem size.

With the separation of inodes and data blocks, there is a *potential disadvantage*: when either the inode table or data block is full, the extra rest space of the file sytem cannot be utilized. Fortunately, some tools can adjust the sizes of either block.

## Hard Link and Symbolic Link

UNIX/Linux supports two ways of creating an *alias* of a file: hard links and symbolic links, which have similar behavior but different implementations.

### Hard Link

* different file names mapped to the same inode
* cannot reference across file systems
* corresponds to system call link

### Symbolic Link

* different file names mapped to different inodes
* can reference across file systems
* corresponds to system call symlink
* actually stores the path to linked file

## Manipulating Files

There are basically two ways to manipulate files in Linux: system calls and library functions. Although the effects can be the same, there are fundamental differences between the two.

**System calls** are interface provided by kernel and are the only interface between programmer and kernel. As only core functionalities should be provided, this interface is as minimal as possible. System calls are documented in the *second section* of `man`.

**Library functions** wrap lower-level operations into a higher-level interface, which is more complex and more programmer-friendly. Library functions depend on system calls. Library functions are documented in the *third section* of `man`.

For example, system calls do not handle buffer, i.e. it reads or write as much as the programmer wants, while IO library functions are implemented with buffer (`FILE*` streams).

### File Descriptor

File descriptor is a small non-negative integer which represents an open file. For each program, three file descriptors are commonly opened by default. They are:

* standard input, `STDIN_FILENO` (0)
    * this is default to the keyboard
* standard ouput, `STDOUT_FILENO` (1)
    * this is default to the monitor
* standard error, `STDERR_FILENO` (2)
    * this is also default to the monitor

### `open` System Call

    int open(const char *path, int flags, mode_t mode);

The call returns the file descriptor of opened file.

#### `flags`

Q: why `flags` is `int`?

A: This technique is called **bit map** and is a both important and commonly used. Bit maps minimizes usage of space and can be ORed (|) together to aggregate composite value.The basic three flags are  `O_RDONLY`, `O_WRONLY` and `O_RDWR` and they are exlucsive to each other. 

If `O_APPEND` flag is set, the append action will be atomic.

#### mode

UNIX uses `mode_t` to store file permission, which is actually an `int` type. While bit operations are efficient they are also esoteric, so in `<sys/stat>` there are macros help determine file permissions, like:

* `S_ISREG`: test if file is regular file
* `S_ISDIR`: test if file is directory
* `S_ISLNK`: test if file is symbolic link

#### File Permission

When creating a file, the actual permission is calculated as follows:

1. get the argument of IO function, say `mode`
1. AND with `umask`: `mode & ~umask`

Note that different functions resolve permission in different ways. `access` checks actual `UID` for permission while `open` checks `EUID` for permission, i.e. if `SUID` is set, the owner's permission is applied.

### Other System Calls

`create`: creates a new file. This system call but it is equivalent to `open` with flag `O_CREAT|O_WRONLY|O_TRUNC`.

`dup`, `dup2`: duplicate file descriptor

`fcntl`: manipulate file descriptor, a vararg function

`stat`: get file status information

`fstat`: `stat` with file descriptor

`lstat`: `stat` only the symbolic link itself

## I/O functions in C Library

The C library provides more comprehensive functions than the bare system calls. The most significant distinction is that C library functions provides auto buffer by default through a standard `FILE *` interface. 

### `FILE*` Stream and File Descriptor

Each `FILE*` has a file descriptor opened underneath and programmers can:

* use `fileno(FILE*)` to get `fd` from `FILE*`
* use `fdopen(fd)` to create `FILE*` stream

There are three predefined `FILE*` streams: `stdin`, `stdout` and `stderr`, corresponding to the three low-level file descriptors respectively.

### Buffering

There are different types of buffer with distinctive behaviors:

* block/full buffer: gets as much as count per read
* line buffer, gets until the new line
* no buffer

To change buffer type, use `setbuf` and `setvbuf`.

Under most circumstances, buffers are handy and efficient than using plain file descriptors, but sometimes they can *cause undesired troubles* as well. For example, when using `printf(fmt)`, if no new line (`\n`) is in the `fmt` string, the characters will first be put in the buffer, otherwise they are output immediately. If you'd like to ensure a flush, use `fflush` to manually force flush.

### Tips for Using Library I/O functions

1. `fread`/`fwrite` can be used to serialize data (`struct`), but why not?

    * in UNIX, files are better off with no structure (as indicated above)
    * portability will suffer

1. Use `fgets` and custom parsing afterwards than `scanf` function family if efficiency is top need. The `scanf` function family just incurs too much overhead.

1. As `stdin` and `stdout` can be redirected, you can use a special file, `/dev/tty` to ensure program read and write from terminal.

    * open it with `read_only` mode for reading 
    * open it with `write_only` mode for writing
