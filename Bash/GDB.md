In order to debug a program effectively, you need to generate debugging information when you compile it. This debugging information is stored in the object file; it describes the data type of each variable or function and the correspondence between source line numbers and addresses in the executable code.

To request debugging information, specify the '-g' option when you run the compile.


# gdb

`gdb`

Start gdb.

`gdb program`

Specify an executable program.

`gdb program core`

Start with both an executable program and a core file specified.

`gdb attach procID`

Attach to a running process. `detach` to release it from GDB control. Detaching the process continues its execution.

`gdb --args gcc -02 -c foo.c`

Pass any arguments after the executable file. This option stops option processing. This will cause gdb to debug gcc, and to set gcc's command-line argumentsto '-02 -c foo.c'.

`gdb --silent`

Disable gdb printing the front material, which describes gdb's non-warranty.

`gdb -ix /home/gdbinit`

gdb 在启动时加载配置文件

`--symbols(-s) file`

Read symbol table from file.

`--core(-c) file`

Use file as a core dump to examine.

`--directory(-d) path`

Add directory to the path to search for source and script file.

If the directory where your program resides is not your current directory, you would better specify an absolute file name to help gdb find the source files.

`-cd directory`

Run gdb using directory as its working directory, instead of the current directory.

`--interpreter(-i)=mi`

Use the interpreter interp for interface with the controlling program or device. This option is meant to be set by programs which communicate with gdb using it as a back end.

`--statistics`

This option causes gdb to print statistics about time and memory usage after it completes each command and returns to the prompt.


# Debug commands

`file`

Specify the executable file.

`run(r)`

Start the program.

If the modification time of your symbol file has changed since the last time gdb read its symbols, gdb discards its symbol table, and reads it again. When it does this, gdb tries to retain your current breakpoints.

`start`

This will automatically find main procedure and set a temporary breakpoint at the beginning of the main procedure and then invoking the 'run' command. `starti` will not skip the elaboration phase.

`info program`

Display information about the status of your program: whether it is running or not, what process it is, and why it stopped.

`set(show) args`

Set or show arguments for your program.

`next(n)`

Advance execution to the next line of the current function.

`step(s) [num]`

Step goes into the next line to be executed in any subroutine.

`finish(fin)`

Continue running until just after function in the selected stack frame returns. Print the returned value (if any).

`until(u) [location]`

Continue running until a source line past the current line, in the current stack frame, is reached. This command is used to avoid single stepping through a loop more than once.

This command can be used to continue execution until it exits the loop.

`advance location`

This command is similar to until, but advance will not skip over recursive function calls, and the target location doesn’t have to be in the sameframe as the current one.

`stepi(si)`

Execute one machine instruction, then stop and return to the debugger.

`print(p)`

Print the variable value.

We can set variable using the p command, since it can print the value of any expression—and that expression can include subroutine calls and assignments. This can modify a variable's value inplace, like `p a=4` will set a to 4.

`print /f`

-   x Regard the bits of the value as an integer, and print the integer in hexadecimal.
-   d Print as integer in signed decimal.
-   u Print as integer in unsigned decimal.
-   o Print as integer in octal.
-   t Print as integer in binary. The letter 't' stands for 'two'.
-   a Print as an address, both absolute in hexadecimal and as an offset from the nearest preceding symbol.
-   c Regard as an integer and print it as a character constant.
-   f Regard the bits of the value as a floating point number and print using typical floating point syntax.
-   s Regard as a string, if possible.
-   r Print using the 'raw' formatting.

`examine(x) addr`

Use the x command to examine memory.

`generate-core-file(gcore) [file]`

Produce a core dump of the inferior process.

`whatis expr`

Print the data type of expression expr.

`list(l)`

Display ten lines of source surrounding the current line.

`info(i)`

This command (abbreviated i) is for describing the state of your program. For example, you can show the arguments passed to a function with info args, list the registers currently in use with info registers, or list the breakpoints you have set with info breakpoints. You can get a complete list of the info sub-commands with help info.

`continue(c)`

Continue executing the program.

`RET`

Repeat the previous GDB command.

`info macro N`

查看宏定义

`macro expand ADD(1)`

拓展宏

`set print elements 0`

设置打印长度无限长

`set print repeats 0`

打印完整的字符，不用 repeat 来进行缩写。gdb 打印内存字符串显示的 '\004' 是八进制表示。

`p (dd::cache::Shared_dictionary_cache)'dd::cache::Shared_dictionary_cache::instance()::s_cache'`

单引号可以帮助打印函数内定义的静态变量

`printf "%s\n", std::string.c_str()`

可以格式化打印字符串

# Breakpoint, watchpoint, catchpoint


## Breakpoint

`break(b) [location]`

Set a breakpoint at the given location, which can specify a function name, a line number, or an address of an instruction. Without location, it will set a breakpoint at the next instruction to be executed in the selected stack frame.

`break ... if cond`

Set a breakpoint with condition cond; evaluate the expression cond each time the breakpoint is reached, and stop only if the value is nonzero—that is, if cond evaluates as true.

`info breakpoints`

Print a table of all breakpoints, watchpoints, and catchpoints set and not
deleted.

`enable(disable) breakpointID`

Enable or disabel breakpoint.

`delete(d) breakpointID`

Delete breakpoint.

`clear location`

Delete any breakpoints set at the specified location.

1.  Location

    `linenum`

    Specifies the line number linenum of the current source file.

    `filename:linenum`

    Specifies the line linenum in the source file filename. If filename is a relative file name, then it will match any source file name with the same trailing components.

    `function`

    Specifies the line that begins the body of the function function. For example, in C, this is the line with the open brace. By default, in C++ and Ada, function is interpreted as specifying all functions named function in all scopes. For C++, this means in all namespaces and classes. For Ada, this means in all packages.

    For example, assuming a program with C++ symbols named A::B::func and B::func, both commands break func and break B::func set a breakpoint on both symbols.

    `filename:function`

    Specifies the line that begins the body of the function function in the file filename. You only need the file name with a function name to avoid ambiguity when there are identically named functions in different source files.


## Watchpoint

You can use a watchpoint to stop execution whenever the value of an expression changes, without having to predict a particular place where this may happen. (This is sometimes called a data breakpoint.) The expression may be as simple as the value of a single variable, or as complex as many variables combined by operators. Examples include:

-   A reference to the value of a single variable.
-   An address cast to an appropriate data type. For example, '\*(int \*)0x12345678' will watch a 4-byte region at the specified address (assuming an int occupies 4 bytes).
-   An arbitrarily complex expression, such as 'a\*b + c/d'. The expression can use any operators valid in the program's native language

`watch [-l|-location] expr [thread thread-id] [mask maskvalue]`

`info watchpoints`


## Catchpoint

`catch event`

Stop when event occurs. The event can be any of the following:

-   throw [regexp]
-   exception


# Backtrace

When your program has stopped, the first thing you need to know is where it stopped and how it got there.

Each time your program performs a function call, information about the call is generated. That information includes the location of the call in your program, the arguments of the call, and the local variables of the function being called. The information is saved in a block of data called a stack frame. The stack frames are allocated in a region of memory called the call stack.

`backtrace(bt) [args]`

Display a stack frame for each active subroutine.

Args can be:

-   n Print only the innermost n frames, where n is a positive number.
-   -n Print only the outermost n frames, where n is a positive number.
-   full Print the values of the local variables also. This can be combined with a number to limit the number of frames shown.

The names where and info stack (abbreviated info s) are additional aliases for backtrace.


## Select a frame

`frame(f) n`

Select frame number n

`frame(f) stack-addr`

Select the frame at address stack-addr.

`up(down) n`

Move n frames up(down) the stack; n defaults to 1.


## Information about a frame

`frame(f)`

When used without any argument, this command does not change which frame is selected, but prints a brief description of the currently selected stack frame.

`info frame(f)`

This command prints a verbose description of the selected stack frame.

`info args`

Print the arguments of the selected frame, each on a separate line.

`info locals`

Print the local variables of the selected frame, each on a separate line. These are all variables (declared either static or automatic) accessible at the point of execution of the selected frame.


# Multiple threads

The gdb thread debugging facility allows you to observe all threads while your program runs—but whenever gdb takes control, one thread in particular is always the focus of debugging. This thread is called the current thread. Debugging commands show program information from the perspective of the current thread.

`info threads`

Display information about one or more threads.

gdb displays for each thread (in this order):

1.  the per-inferior thread number assigned by gdb
2.  the global thread number assigned by gdb, if the '-gid' option was specified.
3.  the target system's thread identifier (systag)
4.  the thread's name, if one is known. A thread can either be named by the user (see thread name, below), or, in some cases, by the program itself.
5.  the current stack frame summary for that thread

`thread thread-id`

Make thread ID thread-id the current thread.

`thread name [name]`

This command assigns a name to the current thread.


# Checkpoint

Returning to a checkpoint effectively undoes everything that has happened in the program since the checkpoint was saved. This includes changes in memory, registers, and even (within some limits) system state. Effectively, it is like going back in time to the moment when the checkpoint was saved.

Thus, if you're stepping thru a program and you think yo're getting close to the poi where things go wrong, you can save a checkpoint. Then, if you accidentally go too far and miss the critical statement, instead of having to restart your program from the beginning, you can just go back to the checkpoint and start again from there.

`checkpoint`

Save a snapshot of the debugged program's current execution state.

`info checkpoints`

List the checkpoints that have been saved in the current debugging session.

`restart checkpoint-id`

Restore the program state that was saved as checkpoint number checkpoint-id.

`delete checkpoint checkpoint-id`

Delete the previously-saved checkpoint identified by checkpoint-id.


# Source file

`list linenum`

Print lines centered around line number linenum in the current source file.

`list function`

Print lines centered around the beginning of function function.

`list -`

Print lines just before the lines last printed.

`list +`

Print lines just after the lines last printed.

`list [location]`

Print lines centered around the line specified by location.

`directory(dir) dirname`

Add directory dirname to the front of the source path. Separate the path with ':'. $cdir:$cwd.

`directory`

Reset the source path to its default value.

`show directory`

Print the source path: show which directories it contains.

`info line location`

Print the starting and ending addresses of the compiled code for source line location.


# Alter execution

`set var x=5 or p x=5`

Set variable.

`jump(j) location`

Resume execution at location. Execution stops again immediately if there is a breakpoint there.

The jump command does not change the current stack frame, or the stack pointer, or the contents of any memory location or any register other than the program counter.

`signal signal`

Resume execution where your program is stopped, but immediately give it the signal signal.

`queue-signal signal`

Queue signal to be delivered immediately to the current thread when execution of the thread resumes.

`return [expression]`

You can cancel execution of a function call with the return command. If you give an expression argument, its value is used as the function’s return value.

`print expr`

Evaluate the expression expr and display the resulting value. The expression may include calls to functions in the program being debugged.

`call expr`

Evaluate the expression expr without displaying void returned values.


# Logging output

`set logging [on|off]`

Enable logging.

`set logging file file`

Change the name of the current logfile. The default logfile is ‘gdb.txt’.

`set logging overwrite [on|off]`

By default, gdb will append to the logfile. Set overwrite if you want set logging on to overwrite the logfile instead.


# Exit

`kill`

Kill the child process in which your program is running under gdb.

`Ctrl-c`

An interrupt (often Ctrl-c) does not exit from gdb, but rather terminates the action of any gdb command that is in progress and returns to gdb command level. It is safe to type the interrupt character at any time because gdb does not allow it to take effect until a time when it is safe.

`quit or Ctrl-d`

Exit gdb.

# GDB Pretty Printer

当 C++ 的某些类型添加 const 或者 & 引用修饰符使用时，可能会出现 pretty printer 报错的行为，这时可以通过`gdb.types.get_basic_type`函数来去除这些修饰符得到最基本的数据类型，见[gdb.types](https://sourceware.org/gdb/onlinedocs/gdb/gdb_002etypes.html#gdb_002etypes)。