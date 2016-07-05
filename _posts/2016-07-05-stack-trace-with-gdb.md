---
layout: post
title: "Stack trace with GDB"
description: "How to find the location where a program has crashed from Linux command line"
category: 
tags: []
---
{% include JB/setup %}

### Stack backtrace from Linux command line

One of the most useful applications of GDB is to get a stack backtrace from
Linux console, when a program crashes e.g. due to a segmentation fault. One
would typically start the program in GDB, run it, and use the `backtrace`
command to print a stack trace.

    % gdb -q my-program
    (gdb) run
    Starting program: /.../my-program
    Program received signal SIGSEGV, Segmentation fault.
    0x00000000004004fd in fail() ()
    (gdb) backtrace
    #0  0x00000000004004fd in fail() ()
    #1  0x0000000000400513 in main ()

In order to get as much information out as possible, the program should be
compiled with debugging information included in the executable. For example:

    % g++ -ggdb my-program.cc -o my-program
    % gdb -q my-program
    Reading symbols from my-program...done.
    (gdb) run
    Starting program: /.../my-program
    Program received signal SIGSEGV, Segmentation fault.
    0x00000000004004fd in fail () at my-program.cc:3
    3           ++*ptr;
    (gdb) backtrace
    #0  0x00000000004004fd in fail () at my-program.cc:3
    #1  0x0000000000400513 in main () at my-program.cc:7

### GDB in batch mode

The above examples expect that you start an interactive session with GDB.
Sometimes it's useful to be able to run a program under GDB non-interactively,
for example when running a program in a compute cluster. For such cases the
batch mode is useful:

    % gdb -q -batch -ex run -ex backtrace my-program
    Program received signal SIGSEGV, Segmentation fault.
    0x00000000004004fd in fail () at my-program.cc:3
    3           ++*ptr;
    #0  0x00000000004004fd in fail () at my-program.cc:3
    #1  0x0000000000400513 in main () at my-program.cc:7

Often one needs to pass some command line arguments to the program that is
debugged. The option `--args` tells GDB that the rest of the command line
specifies the program to be debugged and the arguments to be passed to the
program, i.e.

    % gdb arguments-to-gdb --args my-program arguments-to-my-program

### Multi-threaded programs

By default GDB shows stack trace only for the current thread. When debugging a
multi-threaded program, you may want to use the command `thread apply all
backtrace` to display stack trace for all the threads. Another useful command is
`set print thread-events off`, which disables printing a message every time a
thread starts or exits. Finally, the command `handle <signal> nostop pass` can
be used to instruct GDB not to stop on a signal that the program should be
allowed to handle.

Below is a single command that runs a program under GDB, stops when the program
receives a signal other than SIGALRM or SIGCHLD, and prints the stack backtrace
of all the threads:

    % gdb -q \
          -batch \
          -ex 'set print thread-events off' \
          -ex 'handle SIGALRM nostop pass' \
          -ex 'handle SIGCHLD nostop pass' \
          -ex 'run' \
          -ex 'thread apply all backtrace' \
          --args \
          my-program \
          arguments-to-my-program
