# Processes

In UNIX/Linux, process is the abstraction of dynamic resources.

Advice: read linux source of `struct task_struct` to see what's in a process.

## Process Creating and Terminating

From programmer's view, the entry of a C program is the function `main`, but there is a `crt0.o` linked by linker automatically, in which there exists a `__main` function, the real entrance of a program.

`gcc` compiles C code to ELF, the Executable and Linkable Format, whose name literally indicates how it can be used:

* `gcc -c main.c` would output main.o, which is not executable but linkable
* `gcc main.c` would output  a.out, which is executable

Q: Why the default ouput of an executable file is named `a.out`?

A: `a.out` is abbreviation of _assembly output_, which is used to be in old UNIX format. So while the name still preserves, the internal structure is distinctly different.

A process can get its own process id with call to `getpig()` and get its parent process' id with a call to `getppid()`.

There are five ways to **terminate a process**:

Normal termination:

* `return` from main function
* call `exit` function, a library function from `stdlib.h`
* call `_exit` function, corresponding to a system call from `unistd.h`

Abnormal termination

* call `abort` function
* terminated by a signal

Programmers can add hooks as exit handler with `atexit` and `on_exit` functions. The later registerd function is called earlier.

## `fork` System Call

There are several ways to execute another program in a process, the most handy one maybe the `system` call, which will bring up a new shell process and run the program given. But this solution is inefficiency as it incurs too much burden. The standard practice in Linux programming is use `fork` to duplicate the current process and then use `exec` to replace the duplicated process image. This is only one usage of `fork`.

`fork` creates a child process:

    pid_t fork(void);

this call returns:

* pid of child in parent process
* 0 if in child process
    * child process should know it is the child process at the point of fork
    * can determine execution path on the return value of fork
* -1 if failed

The subprocess is a clone of parent, with independent global and local variables, character buffer and file descriptors. It is common to close some buffers or file descriptors before executing code in child process.

There are also some variants of `fork`:

* `vfork` from `unistd.h` 
* `clone` from `sched.h` 

### Zombie Process

The first process created is the `init` process with PID 0 and all subsequent forked processes are descendants of it. All processes form a tree, and the order of process termination should given special attention.

When parent terminates before the child, the child becomes orphan process. When the child terminates before the parent, `SIGCHLD` signal as well as the `exit code` is sent, and parent can handle this by `wait`/`waitpid` handler.

Warning: If the parent does not handle, the child process would keep waiting parent to handle and thus become zombie. The init process will clean zombie processes regularly. As a result of this, `wait`/`waitpid` can act as a simple process synchronization mechanism.

## `exec` Function Family

The `exec` is a family of functions but correspond to only one system call. They replace the current process image with a new process image. Effects after calling `exec` are:

* pid, ppid, uid, gid, pwd, root dir unchanged
* euid, egid, fd may change:
    * regular programs does not change euid, egid
    * special programs like `sudo`, `passwd` will change euid, egid
    * whether to close fd on `exec` can be set by `fcntl`
        *   `fcntl(fd, F_SETFD, 0)`, this is the default behavor and does not close on `exec`, good for pipe
        *   `fcntl(fd, F_SETFD, 1)`, this closes fd on `exec`
    * opened files are shared by parent and child processes as the external files aren't copied
* uninitialized global variables are assigned value 0

## Environment Variables

Environment variables are a set of dynamic named values that can affect the way running processes will behave on a computer. In programs, environment variables are stored as tables of string: `extern char **environ`.

When using the `exec` functions with `envp` parameter can set environment variables for the replacing process. For example:

    int execve(const char *filename, char * const argv[], char * const envp[]);
