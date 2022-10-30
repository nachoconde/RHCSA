# Shell Scripting

## Basics

### Specifying the Command Interpreter

The first line of a script begins with the notation '#!', commonly referred to as sh-bang or shebang, from the names of those two characters, sharp and bang.

This specific two-byte magic number notation indicates an interpretive script; syntax that follows the notation is the fully qualified filename for the correct command interpreter needed to process this script's lines.

For script files using Bash scripting syntax, the first line of a shell script begins as follows: `#!/bin/bash`

```bash
#!/bin/bash
PLACE=''world''
echo ''hello ${PLACE}!''
```

### Debugging

You can use a debugging technique that will help identify where the script might have failed or did not function as expected.

1. Append the -x option to the “#!/bin/bash” at the beginning of the script: `“#!/bin/bash -x”`,
2. Execute the script as follows `bash -x script.sh`

### Variables

We can use two kinds of variables in a script:

- Locals: Private to the script
- Environment

### Parameters / Arguments

($0) - Represents the command or script itself

($* or $@) → Count of supplied arguments

($#) → all arguments

($$) → PID of the process

($1, $2, $3 . . .) → Argument supplied to a script at the time of its invocation. Beyond 9 we need to enclosed it between curly brackets ${10}

Shift command is used to move arguments one position to the left

Multiple shifts in a single attempt may be performed by furnishing a count of desired shifts to the shift command as an argument. For example, “shift 2” will carry out two shifts, “shift 3” will make three shifts, and so on.

## Control

### For

```bash
for VARIABLE in LIST; do
 COMMAND VARIABLE
done
```

```bash
PLACES_LIST=''Madrid Boston Singapore World''
for PLACE in ${PLACES_LIST}; do
 echo ''hello ${PLACE}!''
done
```

```bash
for LETTER in {A..Z}
do
echo "letter is [$LETTER] "
done
```

```bash
for missions in ` cat missions.txt`
do
 echo &mission
fi
```

Notes

- Attention to the comma `in the for with instructions inside, another way to do is:`$(cat missions.txt)`

### While-loops

```bash
while [ &rocket_launch = 'success' ]
do
     sleep2
done
```

```bash
while true
do
 echo "Press 1 to exit"
 echo "Press 2 to exit"
 read -p "Enter choice:" choice 

if [ $choice -eq 1]
 Exit 0
then
 continue
fi
done
```

### If-Else

Simple if-Else

```bash
if <CONDITION>; then
  <STATEMENT>
elif <CONDITION>; then
  <STATEMENT>
else
  <STATEMENT>
fi
```

### Operators

| Operation | Description |
| --- | --- |
| integer1 -eq (-ne) integer2 | equal / no equal |
| integer1 -lt (-gt) integer2 | less / greater |
| integer1 -le (-ge) integer2 | less /greater or equal |
| string1 = (!=) string2 | identical or not |
| -l string or -z string | string length |
| string or -n string |  |
| ! | Not operator |
| -a or && | AND  |
| -o or || | OR |
| -e | if file exits |
| -d | if file exits and its a directory |
| -s | if files exits and size is greater than 0 |

Notes:

- We should have a space between the comparison statement and also bracket `[ STRING1 = STRING2 ]`
- Equal operator `(=)` is to compare strings
- `-eq`  is to compare integers
- We can use `*` to compare with double brackets `[[ ABC = A*]]`
- To check if two conditions are true we use `[ COND1 ] && [ COND2 ]`

### Exit code / Providing Output

The echo command displays arbitrary text by passing the text as an argument to the command.

By default, the text displays on standard output (STDOUT), but it can also be directed to standard error (STDERR) using output redirection.

 In the following simple Bash script, the echo command displays the message "Hello, world" to STDOUT.

```bash
#!/bin/bash
echo "Hello, world"
echo "ERROR: this is an error." >&2
```

Exist Status = 0 → Success

Exist status >0 → Error

```bash
if [ $rocket_status = "failed" ]
 then 
  exit 1
fi 
```

## Exercises

Create a shell script at `/home/bob/check_dir.sh`  The script should print the line `Directory exists` if the directory `/home/bob/caleston` exists. If not, it should print `Directory not found`

```bash
if [ -d "/home/bob/caleston" ]
then
  echo "Directory exists"
else
  echo "Directory not found"
fi
```

Using the correct command, determine the exit code of the command `ls /home/bob` and redirect the output to the file `/home/bob/lsexit.txt`

```bash
ls /home/bob/
echo $? > /home/bob/lsexit.txt
```

Write a script at `/home/bob/linux-users.sh`

1: Read the user names in the file `/home/bob/users` file.2: Output `$user is a Linux user.` for each user name in the file. `$user` represents the user name, and each output should be appended to a new line.3: The output should be directed to the file `/home/bob/userlist.txt`

```bash
#!/bin/bash
for user in $(cat /home/bob/users.txt)
do
  echo "$user is a Linux user." >> /home/bob/userlist.txt
done
```

Write a script at `/home/bob/ezyum.sh` that appends a single user input to the command `yum`. It should then output `Running the command:` followed by the full command that would be formed using the user input. Lastly, it should run the full command formed with the user input.

```bash
#!/bin/bash
echo Running the command yum $1
yum $1
```
