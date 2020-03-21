# Signals

Signals are a inter-process communication mechanism used in UNIX or UNIX-like systems. Signals are asynchronous and can be used for event-driven programming, which is CPU-friendly than polling.

Programmers can register handlers via `signal` or more modern `sigaction` system calls for handling when specific signals are sent to the running process. **Note** that some signals like `SIGKILL` and `SIGSTOP` can neither be ignored or intercepted.

Definitions of signals can be found in `signal.h` or `man 7 signal`.

## Produce Signals

There are various ways to produce a signal:

* keystroke from terminal
* hardware error
* `kill(2)` function
* `kill(1)` command
* software condition

## Signals and Shell

When performing jobs at shell, three key combinations are frequently used to let shell trigger corresponding signals to running program:

* `Ctrl-C` sends `SIGINT` and by default it causes the process to terminate immediately
* `Ctrl-Z` sends `SIGTSTP` and by default it causes the process to suspend execution
* `Ctrl-\` sends `SIGQUIT` and by default it causes the process to terminate and dump core

**Note** that the default behaviors of the three signals are not guaranteed as programs are free to handle or ignore them.

After a process is suspended by `SIGSTOP`, the user can explicitly wake it up by `kill -s SIGCONT <pid>` or implicilty do it using `fg` or `bg` command.

When a user closes shell, a `SIGHUP` is sent to every child process of the shell and the default handler just terminates the process. `nohup` command can be used to ignore `SIGHUP`.

### Catching Signals in Shell Script

Use `trap` to catch signals sent to the current script:

* `trap` catches system signals
* `trap -l` lists system signals

## Signal Handling

Below are some basic system calls for signal handling, these calls are general and **coarse-grained**.

### `signal`

    #include <signal.h>
    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);

`signal` call registers `handler` for signal of `signum` and returns old handler so that you can restore it later (if you need to).

`handler` can be user-defined or pre-defined, such as:
* `SIG_DFL` - DFL for default
* `SIG_IGN` - IGN for ignore

**Note** that there are some old-fashioned way to use default handlers like `SIG_IGN = (__sighandler_t)1;`, the cast here ensures that the call receives a handler and the call will check its value to make this action safe (just executing function at address 1 would result in disaster!). The black magic here is that pointer is an memory address and in 32-bit system, its value is a 32-bit int.

### `alarm`

    #include <unistd.h>
    unsigned int alarm(unsigned int seconds);

`alarm` sets an alarm clock for delivery of a signal, `SIGALRM`. It returns:

* 0, no previous alarm was set
* or the number of seconds remaining of previous alarm
* in any event any previous set alarm() is canceled! - only one alarm can be set

**Note**: `sleep(3)` may be implemented with `SIGALRM`, so mixing usage of alarm and sleep is a bad idea!

### `pause`

    #include <unistd.h>
    int pause(void);

`pause` waits for any signal to come.

Q: with `alarm` and `pause`, can we simulate `sleep`?

A: yes, but this may cause problems:

1. as `pause` waits for other signals besides `SIGALRM`, when other signals arrive, `pause` is over, which not desirable
1. if the process calling this `customized sleep` is scheduled and put to sleep, and if when it is restored, the time passed exceeds the `seconds` specified, the `SIGALRM` signal is missed and `pause` may wait forever

As `signal` does not take singal mask/signal blocking into consideration, in slightly complex settings trouble may weigh out benefits if it is use. With `struct sigaction` we can do **fine-grained** and more robust signal handling.

### `signaction`

    #include <signal.h>
    struct sigaction {
        handler_t sa_handler; //the same as in signal
        sigset_t sigmask; //signals to block in sa_handler
        sa_flags; //flags to modify signal behavior
        ...
    }
    int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);

### `signsuspend`

As explained above, using `pause` has several vulnerabilities, and the more robust `sigsuspend` is recommended. `sigsuspend` suspends all signals set in `sigmask` and it does this by changing signal mask temporarily.

    #include <signal.h>
    int sigsuspend(const sigset_t *sigmask);
