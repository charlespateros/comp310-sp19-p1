# comp310-sp19-p1
tosh

1 Introduction
For this project you and your partner will be create a new Unix shell, Torero Shell (tosh). tosh can be thought of an extension of your COMP 280 shell, the Tiny Torero Shell (ttsh), with extra features such as I/O redirections and pipes. You will be able to reuse a lot of code from your ttsh but will need to add new code and either modify or remove other parts. Your implementation of tosh will have the following features:
Execution of simple commands (e.g. ls -l)
Running programs in the background (with &)
Built-in commands: cd, exit, history, !num and !!
Support for piping commands (with a single pipe)
Redirection of program input and output
In addition to this functionality, your implementation of tosh must:
Be implemented in C.
Use the execv function, which requires the full path to the executable.
Use the PATH environment variable to locate executables.
Use a signal handler for SIGCHLD to reap children.
Be free of memory-management errors, as confirmed by valgrind.
Use a makefile for compilation.
Be robust to bad/malicious user input.
Gracefully handle errors.
Not use any global variables.
Not assume a “maximum length” of user input.
Have modular design, be efficient, and include production quality commenting.
2 Getting Started
2.1 Accessing Starter Code
The starter code is available on Google drive via blackboard and in the repo. 

2.2 Reusing Code
This project will be an extension of the shell project from COMP 280. As such, it’s suggested that you look over the provided code (or preferably your old code) and decide which pieces can be reused, those that can be adapted, and those that need to be thrown away. There are two major pieces of the COMP 280 shell that you’ll either need to heavily modify or simply throw away:
Command line parsing. Your COMP 280 shell had a simpler syntax that didn’t have to worry about pipes or I/O redirects. You’ll need to make major changes (if not a complete overhaul) to your parser. I recommend the readline library (see details below).
execv vs execvp. Your COMP 280 shell used execvp which automatically searched your system for executables, given just the command name. In this project, you’ll need to use execv, which requires you to give the full path (including directory) to the executable.

2.3 Next Steps
There are two key pieces of code that you will need to write for this project: command line parsing functions and command execution functions. The former takes the user’s input and builds the list of strings to pass as the argv parameter of execv. The parsing functions should be able to handle commands that include IO redirection, built-in commands, pipes, and running commands in the background. The command execution functions handle the tasks you might have guessed: running commands, whether they be simple commands, built-in commands, background commands, etc..
In this project you will be working with one partner so you will want to think about how to most effectively work together. You may be tempted to split the work up so that one of you works on command line parsing functions while the other works on command execution functions. Avoid this temptation. Splitting work this way will lead to one very unhappy group member (i.e. the work for these two parts are not necessarily equal). Apart from this reason, both you and your partner will greatly benefit from having worked on both parts.
My suggestion is that you work on both of these parts together. This doesn’t mean you can’t split off and work independently on some parts but get together frequently to help each other out, check each other’s work, and enjoy the social aspect of programming. Don’t forget that testing and debugging are much more efficient when there are two pairs of eyes staring at the code; don’t forget those pair programming skills you learned in previous courses (especially COMP 280)!
You and your partner should begin by sketching the high-level design of your implementation. It’s a good idea to use a top-down, modular design and start with a set of function prototypes. As always you should start by implementing the simplest functionality possible (e.g. executing a simple command), fully test and debug that functionality, and expand slowly out from there.
A few additional tips before you get started:
A good way to avoid headaches is to use the assert function to test pre- and post-conditions for all the functions you write. Check out the “man page” for assert for details on how to use (i.e. man assert).
Whether you use assert, printf, or some combination for debugging, your final shell shouldn’t contain debugging output. You have several options to deal with this: delete the debugging stuff before turn-in, comment out the asserts and printfs, use C preprocessor macros. The first option is very bad (what if you find a bug at the last minute and need to do more testing?). The second option is fine but can be annoying to manage (“Did I remember to comment out all the lines?”). The third option is preferred as it allows you to turn debugging on and off without modifying any code. Below is an example of the third option.
// "comment out" the next line if you want to disable debug printing
#define DEBUG 1

// ... somewhere later in the code ...
void foo(int my_arg) {
#ifdef DEBUG
    printf("DEBUG: inside foo functionn");
#endif
    // ...
}
Make sure you utilize the gdb debugger and valgrind to help you debug your program.
3 Implementation Details
Your shell is simply a loop that does the following:
Prints the shell prompts (tosh$)
Waits for the user to enter a command (terminated with '\n')
Parses the command that was entered, determines the command name, its arguments, whether it is a built-in function, should be run in the background, etc.
Builds the argv list(s) for execv
Executes the command(s) specified by the user and waits for them to finish unless the user specified that it should be run in the background
Repeats (unless the exit command was executed)
Your shell should directly execute built-in commands cd and exit). All other commands should fork (create) one or more child processes to execute the user’s command, and then wait for the child process(es) to finish before repeating the above loop.
Commands generally follow the form:
commandname arg1 arg2 arg3
The shell checks to see if the command is a built-in. If it is a built-in, the shell invokes the proper function to execute the command directly. Otherwise, the shell parses the command line, creates a child process to execute the command, and waits for the child to finish.
Here are some examples of commands you might run in your shell tosh$ is the command prompt):
tosh$ ls -la        # long listing of curr directory
-rw-------    1 sat  fac         628 Aug 14 11:25 Makefile
-rw-------    1 sat  fac          34 Aug 14 11:21 foo.txt
-rw-------    1 sat  fac       16499 Aug 14 11:26 main.c

tosh$ cat foo 1>  foo.out   # cat foo's contents to file foo.out
tosh$ pwd           # print current working directory
/home/sat/public
tosh$ cd            # cd to HOME directory
tosh$ pwd
/home/sat
tosh$ firefox&      # run firefox in the background
tosh$
You can create a child process with the fork system call then use the wait system call to wait for the child process to exit. fork creates a new process (the child) that shares the same address space as the current process (the parent). Both the parent and the child will continue executing immediately after the call to fork. How do you know if you are the parent or the child? The fork system call returns 0 if you are the child but returns the process id (pid) of the child if you are the parent. The child process will be executing the command so it will be the one making a call to execv. The following idiom outlines how to use fork correctly:
int child_pid = fork();
if (child_pid == -1) {
    // fork failed... handle this error!
    // more code here ...
}
else if (child_pid == 0) {
    // This is the child so execute the command
    // ...
    execv(cmd_name, cmd_argv_list);
    // Check for error here???
}
else {
    // This is the parent. Wait accordingly.
    // ...
}
The parent should use waitpid to wait for the child to exit, as demonstrated in the following example.
// wait until the child with the given pid exits
pid = waitpid(child_pid, &status, 0);
When you call execv, the child process’s image is overlaid with a new process image and starts executing at the beginning point of the new process image (e.g. the main function in a C program). execv does not return unless there was an error—make sure you understand why!
3.1 Reading and Parsing the Command Line
You may use any valid C functions for reading and parsing command line input. The most important consideration is to ensure that your command line parsing is robust in the face of bad input. For example, the user might not enter anything into the prompt, in which case you should simply reprint the command prompt. Another potential problem is if a user enters a malformed command; you should print as informative an error message as possible.
One of the goal of parsing will be to create the arguments to a call to execv
execv(command_name, command_argv_list);
The first argument is a string containing the command name, along with the full path to the executable. IMPORTANT: The COMP280 shell used execvp which didn’t require the full path to the executable. Therefore, instead of “ls”, your string would need to be “/bin/ls”.
The second argument is an array of the following strings:
The command name.
One entry for each argument to the command.
NULL
For example, suppose we wanted to list the contents of the foo directory using the following command:
ls -l foo
Then our call to execvp would be execv("/bin/ls -l foo", arg_list), where arg_list is:
arg_list[0] = “/bin/ls”
arg_list[1] = “-l”
arg_list[2] = “foo”
arg_list[3] = NULL
For reading and parsing, you will want to read about C strings and input functions. Two functions in particular may be useful to you: readline and strtok. If you do use readline, make sure you link in the readline library when compiling, as shown in the following example:
gcc -o tosh tosh.c -lreadline
Make sure you modify your Makefile to include this extra flag (hint: see the CFLAGS variable in your Makefile).
3.1.1 Finding the Path to an Executable
If the user enters a command using its path name, like any of the following examples:
# relative path name examples
./a.out   # a.out is in current dir
../foo    # foo is in parent dir

# absolute path name example
/bin/ls   # an absolute path name
then the path name to the command (that is the first argument to execv) is part of the command line that you read in.
However, if the user enters a command using only its name:
ls
foo
then you need to search the user’s path to find the command executable file and construct the a full path name string to pass to execv:
execv(full_path_to_command_executable, command_argv_list);
To do this, you will use user’s PATH environment variable to try to locate the executable file in the user’s path, and once found you will also test that the file is executable by the user before constructing the full path name string to pass to execv.
Here are some details:
Use the getenv system call to get the value of the PATH environment variable:
char *path = getenv("PATH");
path will be a string that represents an ordered list of directory paths in which to search for the command. It is in the form:
"first_path:second_path:third_path: ..."
For example, if the user’s path is:
/home/comp310/bin:/usr/local/bin:/usr/bin:/usr/sbin
the shell should first look for the executable file (e.g. cat) at /home/comp310/bin/cat. If it is not found there, then it should try /usr/local/bin/cat next, and so on.
(FYI: you can list your PATH environment variable in bash by running “echo $PATH”, and all your environment variables by running the env command)
access: Use this command to check if a file is accessible in some way.
// check if is this file exists and is executable
access(full_path_name_of_file, X_OK);  
Remember: Your shell needs to detect and handle the error of a non-existing command.
3.2 Built-in Commands
The only shell built-in functions you need to handle are cd, exit, history, !num, and !!. For more information on shell built-in functions look at the builtins man page.
Recall that shell built-in functions don’t require fork or exec: instead they are run by the shell process itself.
3.2.1 exit
exit terminates your shell. I recommend printing a friendly goodbye message, e.g. “Thanks for using ttsh. You are the best!”.
3.2.2 cd
To implement the cd command, your shell needs to change the value of the process’ current working directory (cwd). Here are some examples:
cd /usr/bin  # change cwd to /usr/bin
cd           # change cwd to /home/you/, i.e. your home dir
cd ..        # change cwd to the parent of the current directory
After your shell executes a cd command, subsequent executions of pwd and ls by your shell should reflect the change in the cwd that is the result of executing the cd command.
Here are some functions you may find helpful in implementing cd (see their man pages for more details):
getcwd: Gets the absolute path name of the current working directory.
chdir: Change the value of cwd.
getenv: Use this to get the path to your home directory, which is stored in the HOME environment variable. (Try running echo $HOME in bash to see yours).
To implement the cd command, your shell should get the value of its current working directory (cwd) by calling getcwd on start-up. When the user enters the cd command, you must change the current working directory by calling chdir(). Subsequent calls to pwd or ls should reflect the change in the cwd as a result of executing cd.
3.3 history, !!, and !num
This command will list the last n commands entered by the user, where n is either a fixed number (e.g. 10) or an additional parameter (e.g. history 5). You should also add support for the !n command, which will execute command n from your history. Below is an example of how these features should work.
tosh$ history   # list the last 10 commands from history
4   14:56   ls
5   14:56   cd texts/
6   14:57   ls
7   14:57   ls
8   14:57   cat macbeth.txt
9   14:57   cat macbeth.txt | grep damned
10  14:57   pwd
11  14:57   whoami
12  14:57   ls
13  14:57   history
tosh$ !8    # do command 8 from the history (cat macbeth.txt) 
tosh$ !!    # executes the last command again (cat macbeth.txt)
You shouldn’t keep track of all commands has ever issued (otherwise your malicious friend might try filling up your memory by issuing billions of commands!). Instead, you should keep track somewhere on the order of 10 (it should be easy to support as many as you want, but 10 should make it easier to test boundary cases).
A few important things to keep in mind:
Commands from the history list could be any type (built-ins, run in the foreground, commands with pipe, …), thus running a command using !num or !! syntax can result in your shell running any of these different types of commands.
When !num !! are entered, the command line matching that command from the history is added as the most recent command line in the history list (e.g. the string “!9” would never show up in a listing when the user runs history). Try these out in bash to see how they work.
The command number is always increasing, so if you support 50 commands in history and you reach the 51st command, it will be listed as 51, not 1.
Do NOT use the readline library for keeping track of input: rather, you should create a custom data structure for maintaining history. Instead, liberally reuse the code you used in COMP 280 to implement history.
3.4 IO Redirection
Your shell needs to handle I/O re-direction using the 1>, 2>, and < syntax for specifying stdout, stderr, and stdin redirection:
foo 1> foo.out       # redirect foo's stdout to file foo.out
foo 2> foo.err       # redirect foo's stderr to file foo.err
foo 1> foo.out2 2> foo.out2   # redirect foo's stdout/stderr to foo.out2
foo <  foo.in                 # redirect foo's stdin from file foo.out
foo <  foo.in 1> foo.out      # re-direct foo's stdin and stdout
IO redirection using > or >& need not be supported. For example, the following command can be an error in your shell even though it is a valid Unix command:
cat foo > foo.out2 
Each process that is created (forked), gets a copy of its parent’s file descriptor table. Every process has three default file identifiers in its file descriptor table:
stdin (file descriptor 0)
stdout (file descriptor 1)
stderr (file descriptor 2)
The default values for each are the keyboard for stdin, and the terminal display for stdout and stderr. A shell redirects its child process’s IO by manipulating the child’s file identifiers (think carefully about at which point in the fork-exec process this needs to be done). You will need to use the open, close and dup system calls to redirect IO.
3.5 Pipes
A pipe is an interprocess communication (IPC) mechanism for two Unix process running on the same machine. A pipe is a one-way communication channel between the two processes (one process always writes to one end of the pipe, the other process always reads from the other end of the pipe). A pipe is like a temporary file with two open ends, writes to one end by one process can be read from the other end by the another process. However, unlike a regular file, a read from a pipe results in the removal of data that was written by the other process. The pipe system call is used to create a new pipe. It creates two file descriptors, one for the read end of the pipe, the other for the write end of the pipe. One of the communicating processes use the write end of the pipe to send a message to the other communication process who reads from the read end:

When your shell program executes a command with a single pipe like the following:
cat foo.txt | grep -i bar   
cat’s output will be piped to grep’s input. The shell process will fork two processes–one that will exec cat the other that will exec grep–and will wait for both to finish before printing the next shell prompt. Use pipe and IO redirection to set up communication between the two child processes.
The pipe reading process knows when to exit when it reads EOF on its input; any process blocked on a read will unblock when the file is closed. However, if multiple processes have the same file open, only the last close to the file will trigger an EOF. Therefore, when you write programs that create pipes (or open files), you need to be very careful about how many processes have the files open, so that EOF conditions will be triggered.
3.6 Useful Unix System Calls
Here are some Unix system calls that you may find useful, and some examples of how to use them1.
chdir: change the current working directory (needed for cd)
fork-join: create a new child process and wait for it to exit:
int cpid = fork();
if (cpid == -1) { // fork failed, handle the error
    ...
}
else if (cpid == 0) { // the child process
    ...
}
else { // the parent process
    pid = waitpid(cpid, &status, 0);
}
execv: overlay a new process image on calling process
waidpid: Used to wait for a child process to terminate and/or to reap a terminated child. Here are a few examples:
// Wait for specific child process to exit. pid should contain the child id.
pid = waitpid(pid, &status, 0);  

// Reap any child process that has exited (-1) 
// ... but don't wait if none have exited (WNOHANG):
pid = waitpid(-1, &status, WNOHANG);
signal: Register a specific function to handle the designated signal type. Note that the function needs to have a void return type and have exactly one parameter, an int value.
void child_handler(int signum) {
    // code for signal handler goes here...
    // Use a loop that calls waitpid to handle multiple children at once
    // Use the WNOHANG option to waitpid to ensure you don't get stuck
    //  in the handler though...
}

// in some other function...
void foo() {
    // ...

    // register child_handler to as the handler for when we receive a
    //   SIGCHLD signal.
    // SIGCHLD is sent to us automatically when one of our children is
    //   terminated or put to sleep.
    signal(SIGCHLD, child_handler);

    // ...
}
open, close, dup: Useful for IO redirection. For example, to redirect stdout to file foo.txt:
int fid = open("foo.txt", O_WRONLY | O_CREAT, 0666); // open foo.txt
close(1); // close stdout's file descriptor
dup(fid); // duplicate file descriptor, fid
          // the duplicate entry will go into the first 
          // free slot (which happens to be slot 1)
close(fid); // don't need fid open any more
Now when the process write to stdout (i.e. file descriptor 1), the output will go to foo.txt instead of the terminal.
pipe: IPC mechanism for intra-machine processes
int pipe_id[2];
pipe(pipe_id);
read(pipe_id[0], in_buf, len);
write(pipe_id[1], out_buf, len);
perror: Useful for printing out error message for library/system calls such as fork.
pid = fork();
if(pid < 0) { 
    // fork failed for some reason... 
    perror("FORK: n"); // prints "FORK: " followed by an error specific message
}
3.7 Cleaning up your Zombie Hoarde
Your final, correct shell program should reap all exited children and not leave around zombies. However, while working on this project, you may find you are creating a zombie horde. While this is generally fairly harmless, the more zombies you create, the fewer PIDs the system has available. If your horde gets big enough, it can crash your machine. You should therefore deal with your zombie children regularly.
Here’s how you can kill zombies. After running your shell program, run ps in bash to see if you have left any un-reaped zombie children (or any extra instances of your shell program still running):
ps 
233  pts/7   00:00:01  siesta <zombie>
If so, send it (them) a SIGKILL signal to die using either kill or pkill. The following example demonstrates how to use these commands:
# kill the process with PID 233
kill -9 233

# kill all processes with the name siesta
pkill -9 siesta
3.8 Testing Your Shell
In testing your shell, if you are ever unsure about the output of a command line, try running the same command line in bash and see what it does.
Here are some examples of commands that your shell should handle. This is not a complete test suite; you should come up with a much more complete test suite for your shell.
tosh$ 
tosh$  ls 
tosh$  ls -al 
tosh$  cat foo.txt 
tosh$  siesta &
tosh$  cd /usr/bin 
tosh$  ls 
tosh$  cd ../ 
tosh$  pwd 
tosh$  cd 
tosh$  find . -name foo.txt  
tosh$  wc foo.txt
tosh$  wc       blah.txt
tosh$  /usr/bin/ps
tosh$  ps
tosh$  firefox 
tosh$  exit

tosh$  cat foo.txt | more 
tosh$  cat foo.txt | grep blah 
tosh$  cat foo.txt      blah.txt 1>     out.txt 2> out.txt  
tosh$  wc out.txt    
tosh$  cat < foo.txt 1> out2.txt   
tosh$  diff out.txt out2.txt   
tosh$  ls -la yeeha.txt 2> errorout.txt   
tosh$  exit 
The following are some test cases that should cause an error. Remember that your shell should gracefully handle errors by printing an error message and restarting its main loop (i.e. reprint the prompt). Errors should not crash the program or cause it to exit.
tosh$  | 
tosh$  ls 1 >  out 
tosh$  cat foo1> out   
tosh$  1> < 2>  
3.8.1 Creating a Test Script
I highly recommend writing a number of tests and storing them in a file named test-suite.txt. The file should have one command per line, for example:
ls
cat foo.txt
wc bar.txt > meow.txt

cd ..
pwd
exit
You can then take advantage of bash’s IO redirection when running your shell.
./tosh < test-suite.txt
This will have the same effect as if you had manually entered each of these commands, one-by-one. Score!
4 Submission
Submission method is TBD. 
4.1 Required Files
In addition to your source code files, you will need to create the following files.
A plaintext file named README (not README.txt) with the following contents.
The names of you and your partner(s).
A list of features that are either missing, incomplete, or buggy (and an explanation of what bugs they have).
The number of project bucks you are using for the project.
Instructions on how to find the “comments” in your output script file. See below for more details on the output script.
A text file named shell-test.txt showing the output of testing your shell. Make sure your test shows the following:
Simple commands (e.g. cat).
Built-in commands (e.g. cd and exit).
IO redirection.
Pipes.
Launching background commands with &.
Error handling.
If you created one, I would also suggest adding your test-suite.txt file.
4.2 Recording Command line Input/Output with Script
You can use the script utility to record your input and output on the command line. For example, you could try the following:
$> script shell-test-raw.txt
Script started, output file is shell-test-raw.txt
$> ./tosh
tosh$ ls
Makefile main.c shell-text-raw.txt  tosh 
tosh$ exit
good bye!
$> exit

Script done, output file is shell-test-raw.txt
If you open shell-test-raw.txt, you’ll find a bunch of weird characters in there. You can fix this using the following bit of command line magic:
cat shell-test-raw.txt | 
    perl -pe 's/e([^[]]|[.*?[a-zA-Z]|].*?a)//g' | 
    col -b > shell-test.txt
This will put a “cleaned up” version in shell-test.txt.
Extra credit to the first person who posts on Piazza explaining what exactly this command does ;)
You should add comments to the nicely formatted output to describe what you are testing with each command. For example:
#
# Here we show how our shell handles the built-in commands 
#
tosh$  cd
...

#
# Here we show how our shell handles commands with pipes
#
tosh$  cat unicorn.txt | grep rainbow
...

The examples may not match exactly how you need to use them in your shell program.↩
