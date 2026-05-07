# Bash CLI Command Reference

Bash, or the Bourne Again Shell, is a Unix shell and command language written for the GNU Project. It is widely used as the default login shell for most Linux distributions and macOS. Bash is a powerful tool for command-line users and script writers, offering a wide range of built-in commands and utilities. This documentation provides a comprehensive reference for Bash commands, flags, arguments, and usage examples.

## Table of Contents

1. [Introduction to Bash](#introduction-to-bash)
2. [Basic Commands](#basic-commands)
   - [pwd](#pwd)
   - [cd](#cd)
   - [ls](#ls)
   - [echo](#echo)
   - [cat](#cat)
3. [File Management Commands](#file-management-commands)
   - [cp](#cp)
   - [mv](#mv)
   - [rm](#rm)
   - [mkdir](#mkdir)
   - [rmdir](#rmdir)
4. [Text Processing Commands](#text-processing-commands)
   - [grep](#grep)
   - [sed](#sed)
   - [awk](#awk)
5. [System Information Commands](#system-information-commands)
   - [uname](#uname)
   - [df](#df)
   - [top](#top)
6. [Networking Commands](#networking-commands)
   - [ping](#ping)
   - [ifconfig](#ifconfig)
   - [netstat](#netstat)
7. [Process Management Commands](#process-management-commands)
   - [ps](#ps)
   - [kill](#kill)
   - [jobs](#jobs)
8. [Scripting Basics](#scripting-basics)
9. [Advanced Bash Features](#advanced-bash-features)
10. [Conclusion](#conclusion)

## Introduction to Bash

Bash is a command processor that typically runs in a text window where the user types commands that cause actions. Bash can also read commands from a file, called a script. It provides a rich set of programming constructs and a powerful set of text processing tools. Understanding Bash commands and their usage is crucial for system administration, development, and automation tasks.

## Basic Commands

### pwd

#### Description
`pwd` (print working directory) outputs the full pathname of the current working directory.

#### Syntax
```bash
pwd
```

#### Example
```bash
$ pwd
/home/user
```

### cd

#### Description
`cd` (change directory) changes the current directory to a specified directory.

#### Syntax
```bash
cd [DIRECTORY]
```

#### Examples
```bash
$ cd /usr/local
$ cd ../  # Move up one directory
$ cd      # Go to the home directory
```

### ls

#### Description
`ls` lists directory contents.

#### Syntax
```bash
ls [OPTION]... [FILE]...
```

#### Common Options
- `-l`: Use a long listing format.
- `-a`: Include directory entries whose names begin with a dot.
- `-h`: With `-l`, print sizes in human readable format (e.g., 1K, 234M).

#### Examples
```bash
$ ls
Desktop  Documents  Downloads
$ ls -l
total 12
drwxr-xr-x 2 user user 4096 Oct  7 10:00 Desktop
drwxr-xr-x 2 user user 4096 Oct  7 10:00 Documents
drwxr-xr-x 2 user user 4096 Oct  7 10:00 Downloads
```

### echo

#### Description
`echo` displays a line of text.

#### Syntax
```bash
echo [OPTION]... [STRING]...
```

#### Common Options
- `-n`: Do not output the trailing newline.
- `-e`: Enable interpretation of backslash escapes.

#### Examples
```bash
$ echo "Hello, World!"
Hello, World!
$ echo -n "No newline"
No newline$
```

### cat

#### Description
`cat` concatenates and displays files.

#### Syntax
```bash
cat [OPTION]... [FILE]...
```

#### Common Options
- `-n`: Number all output lines.
- `-b`: Number non-blank output lines.

#### Examples
```bash
$ cat file.txt
This is a file.
$ cat file1.txt file2.txt > combined.txt
```

## File Management Commands

### cp

#### Description
`cp` copies files and directories.

#### Syntax
```bash
cp [OPTION]... SOURCE... DIRECTORY
```

#### Common Options
- `-r`: Copy directories recursively.
- `-i`: Prompt before overwrite.
- `-u`: Copy only when the SOURCE file is newer than the destination file or when the destination file is missing.

#### Examples
```bash
$ cp file1.txt file2.txt
$ cp -r dir1/ dir2/
```

### mv

#### Description
`mv` moves or renames files and directories.

#### Syntax
```bash
mv [OPTION]... SOURCE... DIRECTORY
```

#### Common Options
- `-i`: Prompt before overwrite.
- `-u`: Move only when the SOURCE file is newer than the destination file or when the destination file is missing.

#### Examples
```bash
$ mv oldname.txt newname.txt
$ mv file.txt /path/to/destination/
```

### rm

#### Description
`rm` removes files or directories.

#### Syntax
```bash
rm [OPTION]... FILE...
```

#### Common Options
- `-r`: Remove directories and their contents recursively.
- `-f`: Ignore nonexistent files and arguments, never prompt.
- `-i`: Prompt before every removal.

#### Examples
```bash
$ rm file.txt
$ rm -r directory/
```

### mkdir

#### Description
`mkdir` creates directories.

#### Syntax
```bash
mkdir [OPTION]... DIRECTORY...
```

#### Common Options
- `-p`: No error if existing, make parent directories as needed.

#### Examples
```bash
$ mkdir new_directory
$ mkdir -p parent/child/grandchild
```

### rmdir

#### Description
`rmdir` removes empty directories.

#### Syntax
```bash
rmdir [OPTION]... DIRECTORY...
```

#### Example
```bash
$ rmdir empty_directory
```

## Text Processing Commands

### grep

#### Description
`grep` searches for patterns in files.

#### Syntax
```bash
grep [OPTION]... PATTERN [FILE]...
```

#### Common Options
- `-i`: Ignore case distinctions.
- `-r`: Read all files under each directory, recursively.
- `-n`: Prefix each line of output with the line number within its input file.

#### Examples
```bash
$ grep "search_term" file.txt
$ grep -i "pattern" *.txt
```

### sed

#### Description
`sed` is a stream editor for filtering and transforming text.

#### Syntax
```bash
sed [OPTION]... 'SCRIPT' [INPUTFILE]...
```

#### Common Options
- `-e SCRIPT`: Add the script to the commands to be executed.
- `-i[SUFFIX]`: Edit files in place (makes backup if SUFFIX supplied).

#### Examples
```bash
$ sed 's/old/new/g' file.txt
$ sed -i 's/foo/bar/g' file.txt
```

### awk

#### Description
`awk` is a programming language for pattern scanning and processing.

#### Syntax
```bash
awk [OPTIONS] 'program' file...
```

#### Example
```bash
$ awk '{ print $1 }' file.txt
$ awk '/pattern/ { print $0 }' file.txt
```

## System Information Commands

### uname

#### Description
`uname` prints system information.

#### Syntax
```bash
uname [OPTION]...
```

#### Common Options
- `-a`: Print all information.
- `-r`: Print the kernel release.
- `-s`: Print the kernel name.

#### Examples
```bash
$ uname -a
$ uname -r
```

### df

#### Description
`df` reports file system disk space usage.

#### Syntax
```bash
df [OPTION]... [FILE]...
```

#### Common Options
- `-h`: Print sizes in human readable format (e.g., 1K, 234M).
- `-T`: Print file system type.

#### Examples
```bash
$ df -h
$ df -T
```

### top

#### Description
`top` displays Linux tasks.

#### Syntax
```bash
top [OPTION]
```

#### Common Options
- `-b`: Batch mode operation.
- `-n`: Number of iterations.

#### Examples
```bash
$ top
$ top -b -n 1
```

## Networking Commands

### ping

#### Description
`ping` checks the network connectivity to a host.

#### Syntax
```bash
ping [OPTION]... DESTINATION
```

#### Common Options
- `-c`: Stop after sending count ECHO_REQUEST packets.
- `-i`: Wait interval seconds between sending each packet.

#### Examples
```bash
$ ping -c 4 google.com
$ ping -i 2 localhost
```

### ifconfig

#### Description
`ifconfig` configures a network interface.

#### Syntax
```bash
ifconfig [interface]
```

#### Examples
```bash
$ ifconfig
$ ifconfig eth0
```

### netstat

#### Description
`netstat` prints network connections, routing tables, interface statistics, masquerade connections, and multicast memberships.

#### Syntax
```bash
netstat [OPTION]
```

#### Common Options
- `-a`: Show all sockets.
- `-r`: Display the kernel routing tables.
- `-t`: Show TCP connections.

#### Examples
```bash
$ netstat -t
$ netstat -r
```

## Process Management Commands

### ps

#### Description
`ps` reports a snapshot of current processes.

#### Syntax
```bash
ps [OPTION]...
```

#### Common Options
- `-e`: Select all processes.
- `-f`: Full-format listing.

#### Examples
```bash
$ ps -e
$ ps -ef
```

### kill

#### Description
`kill` sends a signal to a process.

#### Syntax
```bash
kill [OPTION] pid
```

#### Common Options
- `-9`: Force kill the process.

#### Examples
```bash
$ kill 1234
$ kill -9 1234
```

### jobs

#### Description
`jobs` displays the status of jobs in the current session.

#### Syntax
```bash
jobs [OPTION]
```

#### Example
```bash
$ jobs
```

## Scripting Basics

Bash scripting allows automation of tasks using the Bash shell. Scripts are text files containing a sequence of commands. Here’s a simple example:

```bash
#!/bin/bash
# This is a comment
echo "Hello, World!"
```

### Variables

Variables store data that can be referenced and manipulated. 

```bash
name="John"
echo "Hello, $name"
```

### Control Structures

Bash supports if statements, loops, and case statements.

#### If Statement
```bash
if [ -f "file.txt" ]; then
    echo "File exists."
else
    echo "File does not exist."
fi
```

#### For Loop
```bash
for i in {1..5}; do
    echo "Iteration $i"
done
```

#### While Loop
```bash
count=1
while [ $count -le 5 ]; do
    echo "Count $count"
    ((count++))
done
```

#### Case Statement
```bash
read -p "Enter a number: " number
case $number in
    1) echo "One";;
    2) echo "Two";;
    *) echo "Other";;
esac
```

## Advanced Bash Features

### Functions

Bash functions allow you to create reusable code blocks.

```bash
function greet() {
    echo "Hello, $1"
}

greet "John"
```

### Arrays

Bash supports one-dimensional arrays.

```bash
arr=("apple" "banana" "cherry")
echo ${arr[0]}   # Outputs: apple
```

### Redirection

Bash supports input and output redirection.

- `>` redirects output to a file, overwriting it.
- `>>` appends output to a file.
- `<` takes input from a file.

```bash
echo "Hello" > file.txt
echo "World" >> file.txt
cat < file.txt
```

### Pipelines

Pipelines use `|` to pass the output of one command as input to another.

```bash
cat file.txt | grep "search_term"
```

### Subshells

Commands in parentheses are executed in a subshell.

```bash
(current_dir=$(pwd))
echo "The current directory is $current_dir"
```

## Conclusion

Bash is an essential tool for anyone working in a Unix-like environment. It offers a wide range of commands and scripting capabilities that make it an invaluable tool for system administration, development, and automation tasks. This comprehensive CLI reference provides a detailed overview of Bash commands, options, and their usage, serving as a foundational resource for both beginners and experienced users.