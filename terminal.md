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
5. Interpreters
6. Background processes
7. Common commands

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

## 
