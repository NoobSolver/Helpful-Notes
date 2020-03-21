# Linux Basic

## Philosophy

As Linux is a UNIX-like operating system, it shares UNIX philosophy. And the most famous one is **KISS**, i.e. Keep It Simple and Stupid.

So when you use Linux as a user, as a programmer, as a contributor, you can always feel that kind of simpleness and unitiesness.

## Kernel

### Processor Rings and Privilege

* Intel x86 processor rings
    * 0-3
    * The smaller, the more privileged
* Linux kernel runs at ring 1
* Linux user programs run at ring 3

### Options to add feature to kernel

1. YES - compiled to kernel
1. NO - not compiled to kernel
1. MODULE - compiled to module, loaded to kernel if needed

### Ways to enter kernek mode

1. Using system call
1. Using soft interrupt

## Devices

### Device Types

#### Block Device

* read unit: block, e.g. 512kbytes
* often used together with cache
* example: hard disk

#### Character Device

* read unit: character
* example: console, keyboard, magnetic tape

### Partition of Storage Device

*Required by Intel-based computers*

* at most four primary partitions
* one primary partition can be extended
* unlimited partitions can exist on an extended partition
    * on linux, the limit is 59

### Master Boot Record (MBR)

should be set up properly in order to boot

Tools:

* Windows: `fdisk /mbr`
    * using chain loading
* Linux: `lilo`, `grub`
    * using a configuration file
    * also supports chain loading

## File and File System

File system can have different meaning in different contexts:

* a sub-module of Linux
* some specific collection of file and certain attributes, such as ext2, ntfs
* a medium of physical device storing logical files
 
### File Systems in Linux:

* VFS: virtual file system (switch), all directories are contained in one, virtual unified "file system" (with mount)
* Supported file systems: ext\*, FAT, NTFS...

### File types in linux:

* `f` - Regular file: no internal structure
* `d` - Directory: a table of contents
* `c` - Character special file
* `b` - Block special file: represent hardware or logical devices
* `p` - FIFO: named pipe, but is created by kernel and accessed as part of the file system, handled by kernel, no actual content 
* `s` - Socket: special file used for inter-process communication
* `l` - Symbolic link: stores file name

### Directories in Linux:

* `/bin`: basic binary files
* `/boot`: boot files
* `/dev`: hardware devices
* `/etc`: configurations
* `/mnt`: mount points
* `/proc`: mirror of the process
* `/root`: home directory of root account
* `/sbin`: executables for administrative tasks, also /usr/sbin and /usr/local/sbin
* `/tmp`: temporary files
* `/usr`: unix system resource, often very large
* `/var`: variable files
* `/lost+found`: recovered files after a power shutdown

### Useful File Commands

* `od`: octal display
* `strings`: view strings of a binary file
* `file`: determine file type
* `lpd`: linear printing daemon
* `wc`: word count

## Programming Linux

### C Programming Language

You can use various programming languages to use Linux system functionalities. But among those languages, `C` is probably the most orthodox one to hack Linux and standard system call interface is provided in C functions. The C programming language is deeply rooted in UNIX culture and it is its creator who created the original UNIX operating system.

### Libraries

Most Linux distributions come with a large warehouse of libraries, in two forms

1. static library
    * in form of `*.a` files 
    * linked to programs at linkage time
    * created by `ar` command
1. dynamic library
    * in form of `*.so` (shared object) files 
    * linked to programs at run time

### Environment Variables

In Linux, each programs runs in a environment, which is in the form of environment variables.

In shell scripts, programmers can use `set` to set environment variables and `export` to make changes to child shells.

In C, the variables are in the form of `char **environ`, which points to an environment table. Programmers can use `getenv` can `putenv` to get and set environment variables.

When a child process is forked and its image if replaced, `execle` and `execve` functions can specify the environment variables the replacing program use.

### `errno`

It is impossible to miss the checking of `errno` if you use Linux system calls, of which most returns `-1` to indicate some error has occurred and sets the `errno` to corresponding values. Programmer can use `strerror` and `perror` to get human-readable information of error occurred:

* `char *strerror(int errno);`
    * defined in `<string.h>`
    * returns a string representation of error
* `void perror(const char *msg);`
    * defined in `<stdio.h>`
    * append user-defined error message and print error

`errno` is defined in `errno.h`. Historically, `errno` is a global value, but modern implementation ensures that `errno` is thread-safe.
