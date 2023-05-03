Download Link: https://assignmentchef.com/product/solved-cs-537-project-2a-unix-shell
<br>
<h2 id="administrivia">Administrivia</h2>

<ul>

 <li><strong>Due Date</strong> by Feb 14, 2020 at 10:00 PM</li>

 <li>Questions: We will be using Piazza for all questions.</li>

 <li>Collaboration: The assignment has to be done by yourself. Copying code (from others) is considered cheating. <a href="http://pages.cs.wisc.edu/~remzi/Classes/537/Spring2018/dontcheat.html">Read this</a> for more info on what is OK and what is not. Please help us all have a good semester by not doing this.</li>

 <li>This project is to be done on the <a href="https://csl.cs.wisc.edu/services/instructional-facilities">lab machines</a>, so you can learn more about programming in C on a typical UNIX-based platform (Linux).</li>

 <li>There is a <strong>quiz on Canvas</strong> that will check if you have read the spec closely. Please remember to also take that!</li>

</ul>

<h2 id="unix-shell">Unix Shell</h2>

In this project, you’ll build a simple Unix shell. The shell is the heart of the command-line interface, and thus is central to the Unix/C programming environment. Mastering use of the shell is necessary to become proficient in this world; knowing how the shell itself is built is the focus of this project.

There are three specific objectives to this assignment:

<ul>

 <li>To further familiarize yourself with the Linux programming environment.</li>

 <li>To learn how processes are created, destroyed, and managed.</li>

 <li>To gain exposure to the necessary functionality in shells.</li>

</ul>

<h2 id="overview">Overview</h2>

In this assignment, you will implement a <em>command line interpreter (CLI)</em> or, as it is more commonly known, a <em>shell</em>. The shell should operate in this basic way: when you type in a command (in response to its prompt), the shell creates a child process that executes the command you entered and then prompts for more user input when it has finished.

The shells you implement will be similar to, but simpler than, the one you run every day in Unix. If you don’t know what shell you are running, it’s probably <code>bash</code>. One thing you should do on your own time is to learn more about your shell, by reading the man pages or other online materials.

<h2 id="program-specifications">Program Specifications</h2>

<h3 id="basic-shell-smash">Basic Shell: <code>smash</code></h3>

Your basic shell, called <code>smash</code> (short for Super Madison Shell, naturally), is basically an interactive loop: it repeatedly prints a prompt <code>smash&gt; </code>(note the space after the greater-than sign), parses the input, executes the command specified on that line of input, and waits for the command to finish. This is repeated until the user types <code>exit</code>. The name of your final executable should be <code>smash</code>.

The shell can be invoked with either no arguments or a single argument; anything else is an error. Here is the no-argument way:

<pre><code>prompt&gt; ./smashsmash&gt; </code></pre>

At this point, <code>smash</code> is running, and ready to accept commands. Type away!

The mode above is called <em>interactive</em> mode, and allows the user to type commands directly. The shell also supports a <em>batch mode</em>, which instead reads input from a batch file and executes commands from therein. Here is how you run the shell with a batch file named <code>batch.txt</code>:

<pre><code>prompt&gt; ./smash batch.txt</code></pre>

One difference between batch and interactive modes: in interactive mode, a prompt is printed (<code>smash&gt; </code>). In batch mode, no prompt should be printed.

You should structure your shell such that it creates a process for each new command (the exception are <em>built-in commands</em>, discussed below). Your basic shell should be able to parse a command and run the program corresponding to the command. For example, if the user types <code>ls -la /tmp</code>, your shell should run the program <code>/bin/ls</code> with the given arguments <code>-la</code> and <code>/tmp</code> (how does the shell know to run <code>/bin/ls</code>? It’s something called the shell <strong>path</strong>; more on this below).

<h2 id="structure">Structure</h2>

<h3 id="basic-shell">Basic Shell</h3>

The shell is very simple (conceptually): it runs in a while loop, repeatedly asking for input to tell it what command to execute. It then executes that command. The loop continues indefinitely, until the user types the built-in command <code>exit</code>, at which point it exits. That’s it!

For reading lines of input, you should use <code>getline()</code>. This allows you to obtain arbitrarily long input lines with ease. Generally, the shell will be run in <em>interactive mode</em>, where the user types a command (one at a time) and the shell acts on it. However, your shell will also support <em>batch mode</em>, in which the shell is given an input file of commands; in this case, the shell should not read user input (from <code>stdin</code>) but rather from this file to get the commands to execute.

In either mode, if you hit the end-of-file marker (EOF), you should call <code>exit(0)</code> and exit gracefully.

To parse the input line into constituent pieces, you might want to use <code>strsep()</code>. Read the man page (carefully) for more details.

To execute commands, look into <code>fork()</code>, <code>exec()</code>, and <code>wait()/waitpid()</code>. See the man pages for these functions, and also read the relevant <a href="http://www.ostep.org/cpu-api.pdf">book chapter</a> for a brief overview.

You will note that there are a variety of commands in the <code>exec</code> family; for this project, you must use <code>execv</code>. You should <strong>not</strong> use the <code>system()</code> library function call to run a command. Remember that if <code>execv()</code> is successful, it will not return; if it does return, there was an error (e.g., the command does not exist). The most challenging part is getting the arguments correctly specified.

<strong>Clarification</strong>: Generally argv[0] for programs is the program name instead of path

<h3 id="paths">Paths</h3>

In our example above, the user typed <code>ls</code> but the shell knew to execute the program <code>/bin/ls</code>. How does your shell know this?

It turns out that the user must specify a <strong>path</strong> variable to describe the set of directories to search for executables; the set of directories that comprise the path are sometimes called the <em>search path</em> of the shell. The path variable contains the list of all directories to search, in order, when the user types a command.

<strong>Important:</strong> Note that the shell itself does not <em>implement</em> <code>ls</code> or other commands (except built-ins). All it does is find those executables in one of the directories specified by <code>path</code> and create a new process to run them.

To check if a particular file exists in a directory and is executable, consider the <code>access()</code> system call. For example, when the user types <code>ls</code>, and path is set to include both <code>/usr/bin</code> and <code>/bin</code>(assuming empty path list at first, <code>/bin</code> is added, then <code>/usr/bin</code> is added), try <code>access("/usr/bin/ls", X_OK)</code>. If that fails, try <code>/bin/ls</code>. If that fails too, it is an error.

Your initial shell path should contain one directory: <code>/bin</code>

Note: Most shells allow you to specify a binary specifically without using a search path, using either <strong>absolute paths</strong> or <strong>relative paths</strong>. For example, a user could type the <strong>absolute path</strong> <code>/bin/ls</code> and execute the <code>ls</code> binary without a search path being needed. A user could also specify a <strong>relative path</strong> which starts with the current working directory and specifies the executable directly, e.g., <code>./main</code>. In this project, you <strong>do not</strong> have to worry about these features.

<h3 id="built-in-commands">Built-in Commands</h3>

Whenever your shell accepts a command, it should check whether the command is a <strong>built-in command</strong> or not. If it is, it should not be executed like other programs. Instead, your shell will invoke your implementation of the built-in command. For example, to implement the <code>exit</code> built-in command, you simply call <code>exit(0);</code> in your smash source code, which then will exit the shell.

In this project, you should implement <code>exit</code>, <code>cd</code>, and <code>path</code> as built-in commands.

<ul>

 <li><code>exit</code>: When the user types <code>exit</code>, your shell should simply call the <code>exit</code> system call with 0 as a parameter. It is an error to pass any arguments to <code>exit</code>.</li>

 <li><code>cd</code>: <code>cd</code> always take one argument (0 or &gt;1 args should be signaled as an error). To change directories, use the <code>chdir()</code> system call with the argument supplied by the user; if <code>chdir</code> fails, that is also an error.</li>

 <li><code>path</code>: The <code>path</code> command takes 1 or more arguments, with each argument separated by whitespace from the others. Three options are supported: <code>add</code>, <code>remove</code>, and <code>clear</code>. <strong>Clarification</strong>: Invalid arguments should be an error.

  <ul>

   <li><code>add</code> accepts 1 path. Your shell should append it to the <em>beginning</em> of the path list. For example, <code>path add /usr/bin</code> results in the path list containing <code>/usr/bin</code> and <code>/bin</code> (notice the order here). Your shell should <em>not</em> report an error if an invalid path is added. It should kindly accept it.</li>

   <li><code>remove</code> accepts 1 path. It searches through the current path list and removes the corresponding one. If the path cannot be found, this is an error.</li>

   <li><code>clear</code> takes no additional argument. It simply removes everything from the path list. If the user sets path to be empty, then the shell should not be able to run any programs (except built-in commands).</li>

  </ul></li>

</ul>

<h3 id="redirection">Redirection</h3>

Many times, a shell user prefers to send the output of a program to a file rather than to the screen. Usually, a shell provides this nice feature with the <code>&gt;</code> character. Formally this is named as redirection of standard output. To make your shell users happy, your shell should also include this feature, but with a slight twist (explained below).

For example, if a user types <code>ls -la /tmp &gt; output</code>, nothing should be printed on the screen. Instead, the standard output of the <code>ls</code> program should be rerouted to the file <code>output</code>. In addition, the standard error output of the program should be rerouted to the file <code>output</code> (the twist is that this is a little different than standard redirection). However, if the program cannot be found (i.e., mistyped <code>pwd</code> as <code>pdd</code>), an error should be reported, but not to be redirected to <code>output</code>.

If the <code>output</code> file exists before you run your program, you should simply overwrite it (after truncating it).

The exact format of redirection is a command (and possibly some arguments) followed by the redirection symbol followed by a filename. Multiple redirection operators or multiple files to the right of the redirection sign are errors. Redirection without a command is also not allowed – an error should be printed out, instead of being redirected.

Note:

<ul>

 <li>Don’t worry about redirection for built-in commands (e.g., we will <strong>not</strong> test what happens when you type <code>path /bin &gt; file</code>).</li>

 <li>Don’t worry about the order of <code>stdout</code> and <code>stderr</code>. In other words, if a process writes to both, the output could be jumbled up. (This is okay!)</li>

</ul>

<h3 id="parallel-commands">Parallel Commands</h3>

Your shell will also allow the user to launch parallel commands. This is accomplished with the ampersand operator as follows:

<pre><code>smash&gt; cmd1 &amp; cmd2 args1 args2 &amp; cmd3 args1</code></pre>

In this case, instead of running <code>cmd1</code> and then waiting for it to finish, your shell should run <code>cmd1</code>, <code>cmd2</code>, and <code>cmd3</code> (each with whatever arguments the user has passed to it) in parallel, <em>before</em> waiting for any of them to complete.

Then, after starting all such processes, you must make sure to use <code>wait()</code> (or <code>waitpid</code>) to wait for them to complete. After all processes are done, return control to the user as usual (or, if in batch mode, move on to the next line).

Note:

<ul>

 <li>Don’t worry about parallel built-in commands (e.g., we will <strong>not</strong> test <code>cd foo &amp; ls -al</code>).</li>

 <li>Redirection should be supported (e.g., <code>cmd1 &gt; output &amp; cmd 2</code>).</li>

 <li>Empty commands are allowed (i.e., <code>cmd &amp;</code>, <code>&amp; cmd</code>).</li>

</ul>

<h3 id="multiple-commands">Multiple Commands</h3>

What if a shell user would like to type in multiple commands in a single line? Sometimes, they might turn in a long list of commands, prepare some popcorns and, wait until all of them to finish. This is supported by semicolons:

<pre><code>smash&gt; cmd1 &amp; cmd2 args1 args2 ; cmd3 args1</code></pre>

Here, your shell runs <code>cmd 1</code> and <code>cmd 2</code> in parallel, like specified above, waits until they complete, and executes <code>cmd 3</code> afterwards.

Note:

<ul>

 <li>Your shell should support multiple built-in commands, such as <code>ls ; cd foo ; ls</code></li>

 <li>Redirection should be supported (e.g., <code>cmd1 &gt; output ; cmd 2</code>).</li>

 <li>Empty commands are allowed (i.e., <code>cmd ;</code>, <code>; cmd &amp;</code>).</li>

</ul>

<h3 id="program-errors">Program Errors</h3>

<strong>The one and only error message.</strong> You should print this one and only error message whenever you encounter an error of any type:

<pre><code>    char error_message[30] = "An error has occurred
";    write(STDERR_FILENO, error_message, strlen(error_message)); </code></pre>

The error message should be printed to stderr (standard error), as shown above.

After most errors, your shell simply <em>continue processing</em> after printing the one and only error message. However, if the shell is invoked with more than one file, or if the shell is passed a file that doesn’t exist, it should exit by calling <code>exit(1)</code>.

There is a difference between errors that your shell catches and those that the program catches. Your shell should catch all the syntax errors specified in this project page. If the syntax of the command looks perfect, you simply run the specified program. If there are any program-related errors (e.g., invalid arguments to <code>ls</code> when you run it, for example), the shell does not have to worry about that (rather, the program will print its own error messages and exit).

For parallel and multiple commands, syntax errors (e.g., <code>ls; &gt; output</code>) or invalid programs names (e.g., a mistyped <code>ls</code>, like <code>lss</code>) should prevent the entire line from executing.

<h3 id="miscellaneous-hints">Miscellaneous Hints</h3>

Remember to get the <strong>basic functionality</strong> of your shell working before worrying about all of the error conditions and end cases. For example, first get a single command running (probably first a command with no arguments, such as <code>ls</code>).

Next, add built-in commands. Then, try working on redirection. Finally, think about parallel and multiple commands. Each of these requires a little more effort on parsing, but each should not be too hard to implement. It is recommended that you separate the process of parsing and execution – parse first, look for syntax errors (if any), and then finally execute the commands.

At some point, you should make sure your code is robust to white space of various kinds, including spaces (<code> </code>) and tabs (<code>t</code>). In general, the user should be able to put variable amounts of white space before and after commands, arguments, and various operators; however, the operators (redirection, parallel commands, and multiple commands) do not require whitespace. For example, your shell should accept commands like:

<pre><code>smash&gt;     ls&amp;    pwd&gt;output     ;cd /usr</code></pre>

Check the return codes of all system calls from the very beginning of your work. This will often catch errors in how you are invoking these new system calls. It’s also just good programming sense.

Beat up your own code! You are the best (and in this case, the only) tester of this code. Throw lots of different inputs at it and make sure the shell behaves well. Good code comes through testing; you must run many different tests to make sure things work as desired. Don’t be gentle – other users certainly won’t be.

Finally, keep versions of your code. More advanced programmers will use a source control system such as git. Minimally, when you get a piece of functionality working, make a copy of your .c file (perhaps a subdirectory with a version number, such as v1, v2, etc.). By keeping older, working versions around, you can comfortably work on adding new functionality, safe in the knowledge you can always go back to an older, working version if need be.

<h3 id="test">Test</h3>

To test your Unix Shell:

<ol>

 <li>Navigate to the directory that contains your smash.c.</li>

 <li>Run the script “runtests” that can be found in /u/c/s/cs537-1/tests/p2a. You may copy it to another location if you prefer.</li>

</ol>

<h3 id="submitting-your-implementation">Submitting Your Implementation</h3>

It is possible to implement the shell in a single <code>.c</code> file, so your handin directory <code>~cs537-1/handin/LOGIN/p2a</code> should only contain these two files:

<ul>

 <li>1 README file named <code>README.md</code></li>

 <li><code>smash.c</code> that contains your source code</li>

</ul>

Use the README file for any explanation you feel would be helpful to someone trying to understand the structure of your code.

We will compile your code via the <code>gcc</code> compiler with flags <code>-Wall</code> and <code>-Werror</code>:

<pre><code>    gcc -Wall -Werror smash.c -o smash</code></pre>

And finally remember to do the quiz on Canvas as well!