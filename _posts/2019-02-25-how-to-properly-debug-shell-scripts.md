---
layout: post
title:  "How to properly debug shell scripts"
date:   2019-02-25 06:30:00 +0800
categories: shell bash debugging
tags: [ shell bash  debugging ]
---

I recently learned the proper way to debug shell scripts. It seems shell
scripting has built-in features that can help developers find problems in their
script. In this post I will be sharing how I debug scripts before and what is
wrong with it while also showing the proper way to do it.

### Contents
1. [Printing commands upon execution.](#print_commands)
2. [Running the entire script on debug mode.](#running_script_on_debug_mode)
3. [Terminating the entire script upon error.](#terminate_script_on_exit)

## <a name="print_commands" />Printing commands upon execution

Before, to debug a script I'm developing, I would do it like this.
```bash
echo "ls pictures"
ls pictures
```
This prints a message before the command is executed. This worked out well for
me before however this means I have to run *echo* for every command that I want
to debug. What if I am debugging multiple consecutive lines of a script?
Suppose I have a script called *foo.sh* with contents like these:
```bash
#!/bin/bash
pwd
whoami
ls pictures
uname
```
If I'm just interested to see if the first three commands, *pwd*, *whoami* and
*ls* are being executed then with my previous way of debugging I would edit its
contents to look like this.
```bash
#!/bin/bash
echo "pwd"
pwd
echo "whoami"
whoami
echo "ls pictures"
ls pictures
uname
```
Thats a lot of *echo* and it looks tedious even though I'm just debugging the
first three commands.

A better way to do this is to execute
```bash
set -x
```
before the commands I want to print. This command prints all the commands upon
execution in the script until it is disabled by executing
```bash
set +x
```
The contents of *foo.sh* would look like this

```bash
#!/bin/bash
set -x # Start printing commands upon execution.
pwd
whoami
ls pictures
set +x # Stop printing commands upon execution.
uname
```
This is better because I only need to execute two commands to print all the
commands being executed and I don't have to execute *echo* which is another
command that may potentially introduce some bugs upon execution of the script.

## <a name="running_script_on_debug_mode" />Running the entire script on Debug mode.

I can run the entire script on debug mode upon its execution from the command
line if I pass the parameter "*-x*" to the script's shebang which is always at
the first line. Executing the command "*set -x*" is no longer necessary. The
script *foo.sh* would look like this.
```bash
#!/bin/bash -x
pwd
whoami
ls pictures
uname
```

## <a name="terminate_script_on_exit" />Terminating the script on error

There are times that if a command the script is executing fails, I want it to
terminate the script immediately, preventing the execution of subsequent
commands.

Suppose that I run the script *foo.sh* and the *pictures* directory doesn't
exist. The command "*ls pictures*" will not execute properly and will result an
error but will still execute the *uname* command. If I don't want to execute
*uname* if an error occurs, I will insert the line below after the *ls* command.
```bash
[ ! $? -eq 0 ] && exit 1
```
and the script to to look like this
```bash
#!/bin/bash
set -x # Start printing commands upon execution.
pwd
whoami
ls pictures
[ ! $? -eq 0 ] && exit 1
uname
set +x # Stop printing commands upon execution.
```
This is how I used to do it but this also has the same problem from the previous
section. Do I have to add this statement after every command that should
terminate the script upon error? That is a tedious task, fortunately I can just
add another parameter "-e" on the *set* command to achieve the same result and 
the script would look like this.
```bash
#!/bin/bash
set -xe # Start printing commands upon execution.
pwd
whoami
ls pictures
set +xe # Stop printing commands upon execution.
uname
```
I can also add "*-e*" as a parameter for the script's shebang if I want the
script to exit upon error of any command like this:
```bash
#!/bin/bash -xe
pwd
whoami
ls pictures
uname
```

That's it! These are the better and proper ways I learned to debug my scripts.
