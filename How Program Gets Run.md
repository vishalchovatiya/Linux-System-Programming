- Do you know how programs get runs behind the screen when you double click on it or you type `./a.out` on shell

- As you know, the standard way to launch an application from shell is to start terminal emulator application & just write the name of the program & pass or not arguments to our program, for example:

```
[vishal@machine Desktop]$ ls --version
ls (GNU coreutils) 8.22
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Richard M. Stallman and David MacKenzie.
```
- This article will give you complete overview of how program gets run behind the scene
- So let's start. The bash shell as well as any program that written with C programming language starts from the main function. If you will look on the source code of the `bash` shell, you will find the main function in the `shell.c` source code file. This function makes many different things before the main thread loop of the bash started to work. For example this function:

    - checks and tries to open /dev/tty;
    - check that shell running in debug mode;
    - parses command line arguments;
    - reads shell environment;
    - loads .bashrc, .profile and other configuration files;
    - and many many more.

- After all of these operations we can see the call of the `reader_loop` function. This function defined in the `eval.c` source code file which made all checks and read the given program name & arguments, it calls the `execute_command` function from the `execute_cmd.c` source code file. The execute_command function through the chain of the functions calls:

```
execute_command
--> execute_command_internal
----> execute_simple_command
------> execute_disk_command
--------> shell_execve
```
makes different checks like do we need to start `subshell`, was it builtin `bash` function or not etc. 
- In the end of this process, the `shell_execve` function calls the `execve` system call which has following signature
```
int execve(const char *filename, char *const argv [], char *const envp[]);
```
