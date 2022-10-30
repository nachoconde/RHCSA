# Bash Shell

# Intro

The bash shell is identified by

- ($) for normal users
- (#) for the root user.

## Variables

- Shell variables: to help run commands or to modify the behavior of the shell
  - Shell variables that are not environment variables can only be used by the shell.

- Environment variables: Automatically copied to programs run from that shell when they start. You can use variables to help make it easier to run a command with a long argument, or to apply a common setting to commands run from that shell.
  - Environment variables can be used by the shell and by programs run from that shell.

```bash
VARIABLE=40
set -> list all variables
echo $VARIABLE
export $VARIABLE ->
unset $VARIABLE 
env --> list of env variables
printenv
```

Using export changes the whole scope of the variable

```bash
MYVAR=1729
export MYVAR=1729
```

The main difference between these two is that the *export* command makes the variable available to all the subsequent commands executed in that shell.

This command does that by setting the export attribute for the shell variable *MYVAR.* **The *export* attribute marks *MYVAR* for automatic export to the environment of the child processes created by the subsequent commands.**

- DISPLAY Stores the hostname or IP address for graphical terminal sessions
- HISTFILE Defines the file for storing the history of executed comm
- HISTSIZE Defines the maximum size for the HISTFILE
- HOME Sets the home directory path
- LOGNAME Retains the login name
- MAIL Contains the path to the user mail directory
- PATH Defines a colon-separated list of directories to be searc when executing a command.
- PPID Holds the identifier number for the parent program
- PS1 Defines the primary command prompt
- PS2 Defines the secondary command prompt
- PWD Stores the current directory location
- SHELL Holds the absolute path to the primary shell file
- TERM Holds the terminal type value
- UID Holds the logged-in user’s UID
- USER Retains the name of the logged-in user

## Input, Output, and Error Redirections

A process uses numbered channels called file descriptors to get input and send output. All processes start with at least three file descriptors.

- Standard input (channel 0) reads input from the keyboard.
- Standard output (channel 1) sends normal output to the terminal.
- Standard error (channel 2) sends error messages to the terminal. If a program opens separate connections to other files, it may use higher-numbered file descriptors.

| NUMBER | CHANNEL | DESCRIPTION | DEFAULT CONNECTION | USAGE |
| --- | --- | --- | --- | --- |
| 0 | stdin | Standard input | Keyboard | read only |
| 1 | stdout | Standard output | Terminal | write only |
| 2 | stderr | Standard error | Terminal | write only |
| 3 | filename | Other files | none | read and/or write |

I/O redirection changes how the process gets its input or output. Instead of getting input from the keyboard, or sending output and errors to the terminal.

| USAGE | DESCRIPTION |
| --- | --- |
| > file | Redirect stdout to overwrite file |
| >> file | Redirect stdout to append file |
| 2>file | Redirect stderr to a file |
| 2> /dev/null | Redirect errors to dev/null |
| > file 2>&1
&> file | Redirects standard output to file and then redirects standard error to the same place as standard output (file) |
| >> file 2>&1
& >> file | Redirect stdout and stderr to append a file |

```bash
#Redirecting errors to files
find /etc -name passwd 2> /tmp/errors
#Erros in separte file than 
find /etc -name passwd > /tmp/output 2> /tmp/errors
#Discard the error
find /etc -name passwd > /tmp/output 2> /dev/null
#Store output and generated errors together
find /etc -name passwd &> /tmp/save-both
#Append output and erros into an existing file
find /etc -name passwd >> /tmp/save-both 2>&1
```

## Pipelines

A pipe connects the standard output of the first command to the standard input of the next command ( `|` )

Pipelines and I/O redirection both manipulate standard output and standard input. Redirection sends standard output to files or gets standard input from files. Pipes send the standard output from one process to the standard input of another process

`ls -t | head -n 10 > /tmp/ten-last-changed-files`

### Tee command

When redirection is combined with a pipeline, the shell sets up the entire pipeline first, then it redirects input/output.

If output redirection is used in the middle of a pipeline, the output will go to the file and not to the next command in the pipeline.

`ls > /tmp/saved-output | less`

In a pipeline

- tee copies its standard input to its standard output
- redirects its standard output to the files named as arguments to the command.

Examples

```bash
ls -l | tee /tmp/saved-output | less
ls -t | head -n 10 | tee /tmp/ten-last-changed-files
```

## History

- `History`  shows commands history
- `!number` Execute command with number n in the history

Variables:

- `HISTFILE`
- `HISTSIZE`
- `HISTFILESIZE`

## Editing at the Command Line

- Ctrl+a: Jump to the beginning
- Ctrl+e: Jump to the end
- Ctrl+u: Clear from cursor to the beginning
- Ctrl+k: Clear from the cursor to the end
- Ctrl+r : Scan the history list for a pattern
- Ctrl+f / Right arrow: Moves the cursor to the right one character at
- Ctrl+b / Left arrow:  Moves the cursor to the left one character

## Alias

Alias substitution allows you to define a shortcut for a lengthy and complex command or a set of command

```bash
alias
alias cp='cp -i'
unalias cp
```

## Metacharacters and Wildcard Characters

The Bash shell has multiple ways of expanding a command line including pattern matching, home directory expansion, string expansion, and variable substitution.

| Pattern | Matches |
| --- | --- |
| * | any characters |
| ? | a single character |
| [abc.. ] | Any one character  |
| [!abc] | Any one character not in the brackets |

### Quoting Mechanisms

metacharacters have special meaning to the shell. In order to use them as regular characters, the bash shell offers three quoting mechanisms to disable their special meaning and allow the shell to treat them as literal characters

1. Prefixing with a Backslash \  `rm \*`
2. Enclosing within Single Quotes ‘’ `echo ‘$LOGNAME’`
3. Enclosing within Double Quotes “”  commands the shell to mask the meaning of all but the backslash (\), dollar sign ($), and single quotes (‘’)

### Tilde expansion

The tilde character (~), matches the current user's home directory.

If it starts a string of characters other than a slash (/), the shell will interpret the string up to that slash as a user name, if one matches, and replace the string with the absolute path to that user's home directory

```bash
ls ~root
ls ~user
cd ~
```

### Command Substitution

Command substitution allows the output of a command to replace the command itself on the command line.

Command substitution occurs when a command is enclosed in parentheses, and preceded by a dollar sign ($).

The $(command) form can nest multiple command expansions inside each other.

## Grep command

`grep` command  helps when finding a pattern in a line, whether in a file or via standard input (STDIN).

Common options for grep are as follows:

- -i: for ignore-case.
- -v: for invert match. This will show all entries that do not match the pattern being searched for.
- -r: for recursive. We can tell grep to search for a pattern in all the files within a directory, while going through all of them (if we have permission).
- -E either . To use variables

```bash
#Find word in the beginning of the file
grep  ^root /etc/passwd
#Find word in the end 
grep bash$ /bin/bash
#Exclude empty lines
grep -v ^$ /etc/login.defs
#Find a word followed by two characters
grep -w acce .. /etc/lvm/lvm.conf
#Piping with AWK
ls -l | grep files | awk '{ print $5}
```

## Controlling Jobs

Job control is a feature of the shell which allows a single shell instance to run and manage multiple commands.

- `sleep 1000 &` Running jobs in backgroud
- `Jobs` display jobs
- `bg` move jobs to background
- `fg %1` moves job 1 to foregound
- `ctrl+z` suspend request

## Bash Shell files

There are two types of startup files:

1. System-wide startup files set the general environment for all users at the time of their login to the system. These files are located in the /etc directory and are maintained by the Linux administrator. System-wide files can be modified to include general environment settings and customizations
    - `/etc/bashrc:` Defines functions and aliases, set umask ...
    - `/etc/profile:` Sets common environment variables such as PATH
    - `/etc/profile.d:` Contains scripts for bash shell users

2. Per-user shell startup files override or modify system default definitions set by the system-wide startup files. These files may be customized by individual users to suit their needs. By default, two such files, in addition to the .bash_logout file, are located in the skeleton directory /etc/skel and are copied into user home directories at the time of user creation.
    - .bashrc: Defines functions and aliases. load global definitions from /etc/bashrc
    - .bash_profile: Sets environment variables and sources

💡 The order in which the system-wide and per-user startup files are executed is important to grasp. The system runs

1. /etc/profile file
2. .bash_profile
3. .bashrc
4. /etc/bashrc
