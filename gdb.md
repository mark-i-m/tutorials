# GDB tutorial

Contents
========
1.  [Starting GDB](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#starting-gdb)
2.  [Starting Execution](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#starting-execution)
3.  [Debugging](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#debugging)
4.  [Stepping through code](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#stepping-through-code)
5.  [Printing data](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#printing-data)
6.  [Convenience variables](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#convenience-variables)
7.  [End Execution](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#end-execution)
8.  [Shortcuts and helpful miscellaneous stuff](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#shortcuts-and-helpful-miscellaneous-stuff)
9.  [TUI](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#tui)
10. [Help](https://github.com/mark-i-m/tutorials/blob/master/gdb.md#help)

Starting GDB
============

Compiling
---------
I will assume you are using gcc/g++, not LLVM-based compilers. These instructions should work equally well for both, but some of the compiler flags might be different.

In order for the debugger to know what assembly lines correspond to what lines of code, you need to compile with debugging info: add the `-g` or `-ggdb` flags to the compilation command. You should also turn off compiler optimizations, as they can do some very intense transformations on the code and confuse the debugger: `-O0`.

Remote Debugging
----------------
You can start gdb without a file by simply executing
```
gdb
```

You will then need to load an executable or symbol table. To load an executable,
```
file <filename>
```

To load a symtable
```
symbol-file <filename>
```

In both cases, you can use the executable file for `<filename>`, but in the second command, only the symbol table will be used.

To attach gdb to a remotely running execution use
```
target remote <host>:<port>
```

For qemu, use `localhost:1234`. Note that you should start the remote execution *before* you try to connect to it with gdb. If the remote execution terminates, you will need to redo this step the next time you run the program.

Local Debugging
---------------
You can also start gdb with a file:
```
gdb <filename>
```

If `<filename>` does not have a symtable, you can specify one as described above. If you compile with `-g`, there should be a symbol table.


Starting execution
==================

If you started gdb with an executable (not a remote execution), use
```
run
```
to start running the program before debugging it.

To run with command line args:
```
run <args>
```

If you attached gdb to a remote execution, you do not need to `run` because it is already running.

Debugging
=========

The first thing you should probably do is set breakpoints and watchpoints. Breakpoints stop exectution at a specific point in the program. Watchpoints stop the program when an expression's value changes.

To set a breakpoint use
```
break <where>
```

`<where>` can take several forms:
- If you want to set a breakpoint at the current line, leave `<where>` blank.
- If you want to set a breakpoint at line x in the current file, use
```
break <x>
```
- If you want to set a breakpoint at line x in file y, use
```
break <y>:<x>
```
- If you want to set a breakpoint at function foo, use
```
break foo
```
- If there are multiple foo's in the program (only in C++, this cannot happen in C) you can disambiguate with the class name:
```
break ClassName::foo
```
- You can also disambiguate overridden functions with parameter types:
```
break foo(HeapNode *)
```

Furthermore, you can set conditional breakpoints:
```
break <where> if <condition>
```
If `<condition>` is true when execution hits `<where>`, the execution stops.

You set a watchpoint with
```
watch <expression>
```
When `<expression>` changes, execution will stop.

To see all watchpoints and breakpoints set, use
```
info b
```
or equivalently
```
info breakpoint
```

This will output a list of breakpoints/watchpoints, where they are in the program or when they occur, if they are conditional, if they are enabled, and a unique number for each one.

To delete a break/watchpoint, use
```
delete <number>
```
where `<number>` is the unique number for the breakpoint you want to delete (which you can get from `info b`).

You can also use
```
clear <function name>
```
```
clear <line number>
```
which delete any break/watchpoints inside of `<function>` or at `<line number>`, respectively.

To clear all breakpoints, use
```
clear
```

Stepping through code
=====================

To step through execution, you can use `step` or `next`. Step goes to the very next line executed. If this means entering a function, gdb will. Next will go to the next line in this function; if there is a function call inside this function, gdb will execute the line without stepping into it.

To step:
```
step
```

To go to next line
```
next
```

Both of these commands can also accept a number, which is the number of lines/steps to take:
```
step 3
next 3
```

There is also a version of step/next that steps by one assembly instruction:
```
stepi
nexti
stepi 3
nexti 3
```

To continue until the next break/watchpoint:
```
continue
```

To print out some lines of code around the line you are stopped at, use
```
list
```
If you call list again without advancing the execution, list will print out the next set of lines in the code. For example if you call list and it prints out lines 0 through 9, then if you call list again without calling next or step, gdb will print out lines 10-19.

List can also take a line number. For example,
```
list 10
list file.c:10
```
will list lines 5 through 15 (ten lines around line 10). The second line in this example shows that a filename can be used to print lines from other files (not the file where execution is stopped).

List can also take a range of lines to print:
```
list 5,15
```
will print lines 5 to 15

In addition, list can take a function name:
```
list foo
list file.c:foo
```
This will print ten lines of code around the beginning of function `foo`. Again, the file name can be used to disambiguate similar-named functions.


Printing data
=============

There are two primary commands for displaying data: `print` and `x` (as in examine). `print` prints the value of some expression, and `x` is used to print the contents of memory.

To print:
```
print <expression>
```
For example:
```
print ((char *)base) + 12
print obj.toString()
```

To examine:
```
x <adr/expression>
```
For example:
```
x (size_t *)this
x/d (size_t *)(this->getNext())
```

`print` and `x` both also take formatting commands:
```
p/FMT <expression>
x/FMT <adr>
```

FMT is one of the following (taken from [GDB's documentation](https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html))


|FMT | Meaning |
|:--:|---------|
|`x` | Regard the bits of the value as an integer, and print the integer in hexadecimal|
|`d` | Print as integer in signed decimal|
|`u` | Print as integer in unsigned decimal|
|`o` | Print as integer in octal|
|`t` | Print as integer in binary (t stands for two)|
|`a` | Print as an address, both absolute in hexadecimal and as an offset from the nearest preceding symbol. You can use this format used to discover where (in what function) an unknown address is located|
|`c` | Regard as an integer and print it as a character constant. This prints both the numerical value and its character representation. The character representation is replaced with the octal escape `\nnn` for characters outside the 7-bit ASCII range. Without this format, GDB displays `char`, `unsigned char`, and `signed char` data as character constants. Single-byte members of vectors are displayed as integer data.|
|`f` | Regard the bits of the value as a floating point number and print using typical floating point syntax.|
|`s` | Regard as a string, if possible. With this format, pointers to single-byte data are displayed as null-terminated strings and arrays of single-byte data are displayed as fixed-length strings. Other values are displayed in their natural types. Without this format, GDB displays pointers to and arrays of `char`, `unsigned char`, and `signed char` as strings. Single-byte members of a vector are displayed as an integer array.|
|`z` | Like `x` formatting, the value is treated as an integer and printed as hexadecimal, but leading zeros are printed to pad the value to the size of the integer type.|
|`r` | Print using the `raw` formatting. By default, GDB will use a Python-based pretty-printer, if one is available (see Pretty Printing in GDB docs). This typically results in a higher-level display of the value’s contents. The `r` format bypasses any Python pretty-printer which might exist.|

Example of `a` FMT:
```
(gdb) p/a 0x54320|
$3 = 0x54320 <_initialize_vx+396>|
```
The command `info symbol 0x54320` yields similar results. See `info symbol`.

Convenience variables
=====================

GDB also defines some standard variables/macros that are helpful. You can uses these anywhere when a gdb command takes an expression:

Some that I have found useful are


| var | Meaning |
|-----|---------|
|`$pc` | The current PC |
|`$eax`... | One for each of the registers, including the stack and base pointers |
|`$_` | The output of the last `x` (examine) command |

You can also create a variable for convenience with
```
set $var = <value>
```
where `var` is the name of the variable you want to create, and value is the `<value>`.

End execution
=============

You can kill the current execution of the program with
```
kill
```

You can quit GDB (and end the program execution) with
```
quit
```

Both of these commands will ask you for confirmation. Note that after GDB has been attached to a process, killing/quiting GDB will kill the process as well.

Shortcuts and helpful miscellaneous stuff
=========================================

Many of the common GDB commands have one-letter shortcuts that are convenient:

| Command | Shortcuts |
|:-------:|:---------:|
|`run`     | `r` |
|`break`   | `b` |
|`list`    | `l` (lowercase L) |
|`next`    | `n` |
|`step`    | `s` |
|`nexti`   | `ni`|
|`stepi`   | `si`|
|`continue`| `c` |
|`delete`  | `d` |
|`kill`    | `k` |
|`quit`    | `q` |

If you press enter without entering a command, the last command will be re-executed.

You can also define new commands for your convenience:
```
define foo
> n
> n
> p/x bar
> end
```
This defines a new gdb command called `foo` which advances execution by two lines and prints the value of `bar` as a hex number.
Each line is a command you want to execute, and the `end` line tells gdb your done defining the command.

TUI
===

GDB also has a curses-based mode, which can show code, assembly, and gdb command prompt at the same time. This is called TUI. It can be toggled from inside gdb with `Ctrl-X, Ctrl-A`. Alternately, it can be started with
```
gdb -tui
```
when you start gdb.

A complete guide to tui can be found [here](https://sourceware.org/gdb/current/onlinedocs/gdb/TUI.html)

TUI treats each part of the screen as a window. The focused window gets control of arrow keys and other special commands, but number typing always goes to the gdb command prompt. You can switch between windows using
```
focus <window>
```

To choose a different layout, use the layout command
```
layout src
layout asm
layout split
```

The `src` layout shows source (c/c++) code and the gdb command prompt. The `asm` layout shows assembly code and the gdb command prompt. The `split` layout shows all three.

Some useful window names are:


| Name | Window |
|:----:|--------|
|`cmd` | gdb command prompt |
|`src` | source (c/c++) code |
|`asm` | assembly code |

There are some caveats to TUI, though. Most importantly, since tui uses curses, if your program has output to `stdout`, it will mess up the screen. You can always refresh using `Ctrl-L`, but it is still pretty annoying. The other important caveat is that sometimes it can be buggy if you toggle tui within GDB without a program executing or if you resize the terminal while GDB is running.

HELP!!!
=======

Finally, gdb has a `help` command that serves as a sort of man page. It can be used to lookup commands to see how they are used and what they do:
```
help <command>
```

For example,
```
help list
```
will display this:
```
List specified function or line.
With no argument, lists ten more lines after or around previous listing.
"list -" lists the ten lines before a previous ten-line listing.
One argument specifies a line, and ten lines are listed around that line.
Two arguments with comma between specify starting and ending lines to list.
Lines can be specified in these ways:
  LINENUM, to list around that line in current file,
  FILE:LINENUM, to list around that line in that file,
  FUNCTION, to list around beginning of that function,
  FILE:FUNCTION, to distinguish among like-named static functions.
  *ADDRESS, to list around the line containing that address.
With two args if one is empty it stands for ten lines away from the other arg.
```
