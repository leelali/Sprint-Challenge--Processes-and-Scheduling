# Sprint Challenge: Processes and Scheduling

## Multiple Choice and Short Answer Questions

Add your answers inline, below, with your pull request.

1. List all of the main states a process may be in at any point in time on a
   standard Unix system. Briefly explain what each of these states mean.

   Processes are "stored" on main memory in most of these states.
   Created: when a process is first created. In this state, the process awaits admission to the "ready" state. Admission will be approved or delayed by scheduler.

   Ready: A "ready" or "waiting" process has been loaded into main memory and is awaiting execution on a CPU. There may have many "ready" processes at any one point of the system's execution, but only one process can be executing at any one time, and others will be waiting for execution. A "ready queue" is used in computer scheduling.

   Running: A process moves into the running state when it is chosen for execution. A process can run in either kernel mode or user mode.

   Blocked: A process transitions to a blocked state when it cannot carry on without an external change in state or envet occuring. For example, a process may block on a call to an I/O device such as a printer.

   Terminated: A process may be terminated, either from the "running" state by completing its execution or by explicitly being killed. The underlying program is no longer executing, but the process remains in the process table as a zombie process until its parent process calls the "wait" system call to read its exit status, at which point the process is removed from the process table, finally ending the process's lifetime. If the parent fails to call "wait", this continues to consume the process table entry, and causes a resource leak.

2. What is a Zombie Process? How does it get created? How does it get destroyed?

   On Unix and Unix-like computer OS, a zombie process is a process that has completed execution (via the "exit" system call) but still has an entry in the process table: it is a process in the "Terminated state". This occurs for child processes, where the entry is still needed to allow the parent process to read its child's exit status: once the exit status is read via the "wait" system call, the zombie's entry is removed from the process table and it is said to be "reaped". A child process always first becomes a zombie before being removed from the resource table. In most cases, under normal system operation zombies are immediately waited on by their parent and then reaped by the system - processes that stay zombies for a long time are generally an error and cause a resource leak. Also, unlike normal processes, the "kill" command has no effect on a zombie process. [notes: Zombie processes should not be confused with "orphan processes", which is a process that is still executing, but whose parent has died]

3. Describe the job of the Scheduler in the OS in general.

   The scheduler is an operating system module that selects the next jobs to be admitted into the system and the next process to run.

   Goals of scheduler: 
   1. maximizing throughput (the total amount of work completed per time unit);
   2. minimizing wait time (time from work becoming enabled until the first point it begins execution on rescources);
   3. minimizing latency or response time;
   4. maximizing fairness.
   In practice, these goals often conflict, thus a scheduler will implement a suitable compromise.

   There are three types of schedulers:
   1. Long-Term scheduler
   2. Short-Term scheduler
   3. Medium-Term scheduler

4. Describe the benefits of the MLFQ over a plain Round-Robin scheduler.

   MLFQ is multilevel feedback queue scheduling algorithm. MLFQ scheduler assign processes to queues, and allow a process to move between queues. If a process uses too much CPU time it will be moved to a lower-priority queue. In addition, a process that waits too long in a lower-priority queue may be moved to a higher priority queue. 

   Round-robin(RR): time slices are assigned to each process in equal portions and in circular order, handling all processes without priority.

## Programming Exercise: The Lambda School Shell (`lssh`)

This program implements a new shell that you can use to run commands from in
Unix, similar to bash!

At the end of the day, you should be able to run your shell, the run commands within it like so:

```
[bash]$ ./lssh 
lambda-shell$ ls -l
total 32
-rwxr-xr-x  1 beej  staff  9108 Mar 15 13:28 lssh
-rw-r--r--  1 beej  staff  2772 Mar 15 13:27 lssh.c
lambda-shell$ head lssh.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

#define PROMPT "lambda-shell$ "

#define MAX_TOKENS 100
#define COMMANDLINE_BUFSIZE 1024
#define DEBUG 0  // Set to 1 to turn on some debugging output
lambda-shell$ exit
[bash]$ 
```

General attack is to:

* Loop until the user exits.
  * Print a prompt.
  * Read a line of input from the keyboard.
  * Parse that line down into individual space-separated parts of the command.
  * Fork a child process to run the new command.
  * Exec the command in the child process.
  * Parent process waits for child to complete.

Some of the shell is already written, but you have to write the guts of it.

Examine the `lssh.c` file and:

* Determine which parts of the program do what.
* Read the header comments.
* Find the parts you need to implement.
* Come up with an individual plan of attack for each of those parts.
* Determine how to exit the shell.
* What happens if you build it?
* What happens if you run it?

Resist the urge to start coding until you've done all that and read this
complete challenge.

Hint: you probably want one of the `exec` variants that has the `p` suffix on it
so that the `PATH` will be searched for the file you're trying to run. Choose
your `exec` variant carefully. Some will be hard-to-impossible to use. Some will
be easy.

When you get this working, _you will have your own shell_. It's not as
full-featured as bash, by any means, but it is a legitimate shell and the core
is the same as any other shell.

If you finish early, look at extra credit to start implementing more features.

### Extra Credit: Change Directories with `cd`

In bash, you can change directories with the built-in `cd` command.

Each process keep track of which directory it is running in, and the shell is no
exception. Each process can change its current directory, as well.

> Why is `cd` built into the shell? Why can't it run as an external command?

Because it is built in, you should check to see if the user entered `cd` in
`args[0]` _before_ running the command. And if they did, you should
short-circuit the rest of the main loop with a `continue` statement.

> Look at the implementation of the built-in `exit` command for inspiration.

You can use the program `pwd` to see what directory you are in.

Example run:

```
lambda-shell$ pwd
/Users/example
lambda-shell$ cd src
lambda-shell$ pwd
/Users/example/src
lambda-shell$ cd ..
lambda-shell$ pwd
/Users/example
lambda-shell$ cd foobar
chdir: No such file or directory
lambda-shell$ 
```

If the user entered `cd` as the first argument:

1. Check to make sure they've entered 2 total arguments
2. Run the system call `chdir()` on the second argument to change directories
3. Error check the result of `chdir()`. If it returns `-1`, meaning an error
   occurred, you can print out an error message with:
   ```
   perror("chdir"); // #include <errno.h> to use this
   ```
4. Execute a `continue` statement to short-circuit the rest of the main loop.

Note that `.` and `..` are actual directories. You don't need to write any
special case code to handle them.

### Extra Credit: Background Tasks

In bash, you can run a program in the background by adding an `&` after the
command.

```
ls -la &
```

Add similar functionality to `lssh` by checking to see if the last argument is
an `&`.

If it is:

1. Strip the `&` off the `args` (by setting that pointer to `NULL`).
2. Run the command in the child as usual.
3. Prevent the parent from `wait()`ing for the child to complete. Just give a
   new prompt immediately. The child will continue to run in the background.

Note that you might get weird output when doing this, like the prompt might
appear before the program completes, or not at all if the program's output
overwrites it. If it looks like it hangs at the end, just hit `RETURN` to get
another prompt back.


### Extra Credit: File Redirection

In bash, you can redirect the output of a program into a file with `>`. This
creates a new file and puts the output of the command in there instead of
writing it to the screen.

```
ls -l > foo.txt
```

Check the `args` array for a `>`. If it's there:

1. Get the output file name from the next element in `args`.
2. Strip everything out of `args` from the `>` on. (Set the `args` element with
   the `>` in it to `NULL`).
3. In the child process:
    1. `open()` the file for output. Store the resultant file descriptor in a
       variable `fd`.
    2. Use the `dup2()` system call to make `stdout` (file descriptor `1`) refer
       to the newly-opened file instead:

	    ```c
		int fd = open(...
		dup2(fd, 1);  // Now stdout goes to the file instead
		```
	3. `exec` the program like normal.

### Extra Credit: Pipes

In bash, you can pipe the output of one command into the input of another with
the `|` symbol.

For example, this takes the output of `ls -l` and uses it as the input for `wc
-l` (which counts the number of lines in the input):

```
ls -l | wc -l
```

This uses the `pipe()` system call.

Use a process similar to the above extra credit challenges, along with [this
description of how to implement pipes in
C](https://github.com/LambdaSchool/CS-Wiki/wiki/How-Unix-Pipes-are-Implemented)
to get pipes implemented in `lssh`.