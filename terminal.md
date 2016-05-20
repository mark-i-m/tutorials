# \*nix Terminals (for beginners)

This is tutorial about Unix/Linux style terminals, not Windows `cmd` style
command prompts. IMHO, \*nix terminals are far more powerful. Almost in
admission of this, Windows 10 is adding kernel support for running [*bash* on
Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about)!

Moreover, I will focus on `bash` because this is the terminal I most familiar
with, it's easy for beginners, and it comes preinstalled on most Linux systems
already. If this means nothing to you, that's ok.

## Contents

1. What is a terminal (emulator)?
2. The basics
3. `stdin`, `stdout`, `stderr`
4. Pipes
5. Background processes
6. Interpreters
7. Scripting
8. Variables and Other Useful Tips
9. Common commands

## What is a terminal (emulator)?

A terminal is basically just the combination of a keyboard and a screen. In the
old days, when you used a computer, that's all there was: just a keyboard and a
command prompt on the screen. The prompt allows users to enter commands, and it
returns to them feedback about the execution of their commands.

Today, technically, we don't use terminals, per se. We use programs that provide
this functionality: *terminal emulators*. That's all there is to it.

There have been a wide variety of emulators over the years. The long ancestry of
shells starts with the original Unix shell `/bin/sh`. Later, Stephen Bourne
introduced the Bourne shell. Parallelly developed were the Korn shell (`ksh`)
and the `csh` (pronounced like "sea shell"). From Bourne shell came Bourne Again
Shell `bash`, and from `csh` came `zsh`.

Today, `bash` is extremely popular (not to say that others like `zsh` aren't),
and my experience is mostly with `bash`, so I will focus on `bash`. Different
shells have different behaviors and syntax, but they all more or less have
similar basic functionality.

## The basics

To discuss the basics, let me first back up and explain how a program is run in
traditional Unix systems.

The operating system exposes two primary system calls for creating and running
new processes: `fork` and `exec`. `fork` makes an exact clone of this process.
`exec` replaces this process with the specified program.

The `exec` system call takes some parameters:

1. The name of the file to execute
2. A list of arguments
3. A list of environment variables

Let's look at each of these.

The filename refers to some file in the filesystem that the operating system can
reach and that has executable permissions. For example, `/bin/ls` is an
executable that lists all of the files and directories in the current directory.

Obviously, we need some way for the user to provide information to the program
being `exec`ed. This is what the arguments and environment variables are for.
Each is a list of ASCII strings. The arguments list can contain any arbitrary
strings, but the first argument is always the name of the program being
`exec`ed, by convention. The environment variables list is a list of strings of
format `name=value`, for non-empty `name`.

So what does this have to do with shells? Well, to start a new program, all a
shell has to do is `fork` itself, then `exec` the desired program. (Linux
systems are a bit different under the hood, but this still works).

Let's start from the top. If you start a new terminal session, you should be
presented with a prompt of some sort. Usually, Unix shells have the `>` prompt,
while `bash` tends towards `$` or `#` (if you are the super-user). Often, this
symbol is precede by some useful information, such as the name of the machine
and the current directory. For example:

```
mark@demo ~/demo1/ $
```

Here, `mark` is my username. `demo` is the name of my machine. `~/demo1/` is the
current working directory (in `bash`, `~` is an abreviation for the current
user's home directory).

We can enter a command as such:

```
mark@demo ~/demo1/ $ ls -l
total 0
-rw-rw-r-- 1 mark mark 0 May 20 00:02 demo_file.txt
```

We just issued the `ls` command with argument `-l`. `ls` stands for "list", and
`-l` is a *flag* that tells less use "long listing format". Notice that `-l` is
an *argument* passed to `ls`. Sound familiar? That's right! `bash` just called
`fork`. The new bash process called `exec` passing `/bin/ls` as the executable,
and the list `['ls', '-l']` (to use python syntax) as the arguments list. It
also passed a whole bunch of envirnment variable that were passed to the
original `bash` process when it started. Generally, you don't need to worry
about these.

The `ls` program generated some output. It appears that there is a single file
in `~/demo1/` called `demo_file.txt`. `ls` gives some info about its file
permissions, owner, group, size, and date and time of creation.

That's basically it. All commands have similar format. But there are some other
really useful features that make shells so powerful.

## `stdin`, `stdout`, `stderr`

Let's try something new:

```
mark@demo ~/demo1/ $ cat demo_file.txt
mark@demo ~/demo1/ $
```

As you probably guessed, we just executed `/bin/cat` with arguments `['cat',
'demo_file.txt']`. But it doesn't seem to do anything...

This is because `cat` simply dumps the contents of the file to the terminal, and
it happens that `demo_file.txt` is completely empty. Let's put something into
it.

(As a useful sidenote, you might be getting tired of retyping `demo_file.txt`
every time. Most shells now offer tab completion. For example, type `demo_`
followed by tab. If `demo_` is enough for `bash` to figure out what you want, it
will fill in the rest. Otherwise, it will print a list of ambiguous options. You
can add some more characters and try again. This can really speed things up!)

```
mark@demo ~/demo1/ $ echo 'Hello, world!' > demo_file.txt
mark@demo ~/demo1/ $ cat demo_file.txt
Hello, world!
mark@demo ~/demo1/ $
```

So how does this work? You might guess that we are running `/bin/echo` with
`['echo', '\'Hello, world!\'', '>', 'demo_file.txt']`. Nope!

Let me backup and explain another aspect of Unix. Each process has a numbered
list of file descriptors. The process can do I/O via these files. The first
three file descriptors (0, 1, and 2) are special. 0 is `stdin`; 1 is `stdout`;
and 2 is `stderr`. `stdin` is the standard default file for getting input at
runtime. `stdout` is the standard default file for printing output. `stderr` is
the standard default file for printing errors.

Usually, the three refer to the terminal that spawned the process. That is,
`stdin` gets input from the keyboard, and `stdout` and `stderr` print to the
screen. If we want to get input from or send output to somewhere else, we can
simply change what the appropriate file descriptor points to.

As a matter of fact, this is exactly what the `bash` syntax `>` does. In our
example, if we simply run

```
mark@demo ~/demo1/ $ echo 'Hello, world!'
Hello, world!
```

We can see that `/bin/echo` simply outputs whatever its arguments are, and we
gave it the argument `'Hello, world!'` (the single quotes make this a single
string, as opposed to two space-separated strings). By using `>` we redirected
`stdout` for `echo` into the file `demo_file.txt`. Voila!

You can also redirect `stdin`:

```
mark@demo ~/demo1/ $ grep 'Hello' < demo_file.txt
Hello, world!
```

`grep` is a program that evaluates regular expressions on its `stdin`. In this
case, it is looking for the string `Hello`. We redirect `stdin` for grep to be
`demo_file.txt`. `grep` finds "Hello" in the phrase "Hello, world!" on the first
line, so it outputs this line.

As you might expect, we can also redirect `stderr`:

```
mark@demo ~/demo1/ $ ls nonsense 2> error.txt
mark@demo ~/demo1/ $ cat error.txt
ls: cannot access 'nonsense': No such file or directory
```

Here, we are telling `ls` to tell us the contents of the `nonsense` directory.
However, this directory does not exist, and `ls` promptly prints an error
message telling us this. We redirected this error into `error.txt`.

Finally, we can actually set `stdout` and `stderr` to the same location. This
can be useful for logging:

```
mark@demo ~/demo1/ $ ls nonsense existing > out.txt 2>&1
mark@demo ~/demo1/ $ cat out.txt
ls: cannot access 'nonsense': No such file or directory
existing/:
```

The `2>&1` tells `bash` to send `stderr` to `stdout`. We could equally validly
do `1>&2` to send `stdout` to `stderr`. `ls` tells us that `nonsense` does not
exist and that `existing` is an empty directory. All of this is captured in
`out.txt`.

## Pipes

So let's say that I want to check if the current directory has a file in it. One
option would be to do `ls` and scan through the results. This works if the
directory is small, but what if there are hundreds of files?

Let's try using redirection.

```
mark@demo ~/demo1/ $ ls > /tmp/ls_output
mark@demo ~/demo1/ $ grep 'some_file' /tmp/ls_output
```

We are running `ls` in the current directory and sending the output to
`/tmp/ls_output`. Then, we use `grep` to look for `some_file` in the output. (A
couple of quick notes: (1) Notice we ran `grep` passing `/tmp/ls_output` as an
argument rather than piping, and (2) in reality, we could have just done `ls
some_file`, which returns an error if `some_file` does not exist).

This is a bit clunky, though. We don't really care about `/tmp/ls_output` except
that we want to pass the output to `grep`. Thankfully, we can do just that with
*pipes*!

```
mark@demo ~/demo1/ $ ls | grep 'some_file'
```

The `|` (called a pipe) syntax tells `bash` to redirect the `stdout` of `ls`
into the `stdin` of `grep`. And we're done.

This example might seem a bit contrived, but consider that we can string
multiple pipes together on the same command. Unix is built on the notion that
tools should do exactly one thing, and do it really well. We can then use
multiple simple tools to break down a complex problem. For a good example, see
[here](http://unix.stackexchange.com/questions/30759/whats-a-good-example-of-piping-commands-together).

## Background processes

Let's say I am running a python script on my terminal, and I want to quickly
check the contents of the current directory. But I can't because the script has
control of the terminal right now...

One option would be to go to the directory in another terminal. For the purpose
of illustration, I won't do that. Instead, I will hit `Ctrl-z`. This stops the
python script and returns control of the terminal to me. I can run other
commands now. When I want to return to running the script, I can use the `fg`
(foreground) command to return the script to the foreground.

```
mark@demo ~/demo1/ $ ./long.py
^Z
[1]+ Stopped                    ./long.py
mark@demo ~/demo1/ $ ls
...
mark@demo ~/demo1/ $ fg
./long.py
^C
```

Ok, there is a lot to go through here. First, let me address `Ctrl-z` and
`Ctrl-c`. These are keyboard shortcuts that tell the operating system to send
*signals* to the current process. `Ctrl-z` is caught by `bash`, and `bash` then
causes the python process to stop running for the moment. `Ctrl-c` goes straight
to the python script. Usually, `Ctrl-c` kills a process, but the program being
run can choose to catch the signal. This is usually done to avoid corrupting
data.

Generally, you should be very careful when using `Ctrl-z` and `Ctrl-c` because
you can corrupt data by interrupting at the wrong time!

So what does `fg` do? As you probably guessed, it tells `bash` to switch control
back over to the python process and let python pick up where it paused. But what
if we actually wanted python to keep running in the background?

```
mark@demo ~/demo1/ $ ./long.py
^Z
[1]+ Stopped                    ./long.py
mark@demo ~/demo1/ $ bg
[1]+ ./long.py &
mark@demo ~/demo1/ $ ls
...
```

The `bg` command tells `bash` to allow the python process to continue but in the
background.

As a matter of fact, we could have started the python process in the background
in the first place:

```
mark@demo ~/demo1/ $ ./long.py &
[1] 13976
mark@demo ~/demo1/ $
```

`bash` starts the process and immediately presents a new prompt.

## Interpreters

You may be wondering how we could just run `./long.py` in the previous example.
After all, python files are not machine code... you cannot just run them; you
need an interpreter. This is true. So what is the secret?

```
mark@demo ~/demo1/ $ cat long.py
#!/usr/bin/env python2
...
```

If the first two characters of a file (with executable permissions) are `#!` (I
have a friend who pronounces this "sha-bang"), the shell will assume this line
names an interpreter for the rest of the file. In this case, `bash` will not
`exec` `long.py`. Instead, it will `exec` `/usr/bin/env` with the argument
`python2` and redirect `stdin` to the rest of `long.py`.

This turns out to be a useful trick for cutting down on typing, especially if
you are providing someone else with a quick utility script. And speaking of
scripting...

## Scripting


