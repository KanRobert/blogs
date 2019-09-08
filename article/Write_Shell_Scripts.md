# Reference
1. [TLCL chapter 25](http://billie66.github.io/TLCL/book/chap25.html)
1. [declare usage](https://www.computerhope.com/unix/bash/declare.htm)
1. [insert Tab when expandtab option is ON](https://stackoverflow.com/questions/4781070/how-to-insert-tab-character-when-expandtab-option-is-on-in-vim)

# What is a shell script?
A shell script is an executable file containing a series of command. The shell reads the file and carries out the commands as if they are entered on the command line.

# How to create and run a shell script?
1. **Write a script.** Shell scripts are text files, so we need a text editor, such as vim, to write them.
1. **Make the script executable.** The system does not allow any old text to be treated as a program for good reason. We need to set the script file's permission to allow execution. 
1. **Put the script somewhere the shell can find it.** The shell auomatically searches certain directories for executable files when no explicit pathname is specified. For convenience, we will put our scripts in these directories.

## Write an extremely simple script
```bash
#!/bin/bash
# This is our first script hello_world.
echo 'Hello World!'
```
Everthing from the # symbol onward on the line is ignored, namely the second line is a script. The last line is an echo command with a string argument.

The first line is a little mysterious. The #! character sequnece is a special construct called shebang. The shebang is used to tell the system the name of the interpreter that should be used to execute the following commands.

## Make the script executable
This is easily done using chmod:
```bash
$ ls -l hello_world
-rw-r--r-- 1 owner group 72 Sep  8 13:28 hello_word
$ chmod 755 hello_world
$ ls -l hello_world
-rwxr-xr-x 1 owner group 72 Sep  8 13:28 hello_world 
```
There are two common permission settings for scripts, 755 for scripts that everyone can execute, and 700 for scripts tha only the owner can execute. Note that scripts must be readable in order to be executed. You can use python to convert the octal to binary:
```python
>>> bin(0o755)
'0b111101101'
```

## Put the script somewhere the shell can find it
In order for the script to run, we must precede the script name with an explicit path. If we don't, we get this:
```bash
$ hello_word
bash: hello_world: command not found
```
Why is this? The PATH environment variable determines where the system searches for executable programs and it contains a colon-separated list of directories. We can view the contents of PATH:
```bash
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```
This is why the system knows execute `/usr/bin/ls` when we type `ls` at the command line. So if we create a directory, place our script within it and add the path of thw directory to PATH variable, hello_world should work like other programs:
```bash
$ mkdir -p ~/bin
$ mv hello_world ~/bin
$ export PATH=$PATH:~/bin
$ hello_world
Hello World!
```
And so it does.

After this change is made, it will take effect only in current session. It's easy to make it take effect in each new terminal session:
```
$ echo 'export PATH=$PATH:~/bin' >> ~/.bashrc
$ . ~/.bashrc
```
The >> operator is to append output to the end of the file. The dot(.) command is a synonym for the source command, a shell builtin which reads a specified file of shell commands and treats it like input from the keyboard.

# Variable
## Create a variable
How can we create a variable? Simple, we just use it. When the shell encounters a variable, it automatically creates it:
```bash
$ foo='yes'
$ echo $foo
yes
$ echo $fool

$
```
When we display the value of the variable name misspelled as "fool" and get a blank result. This is because the shell happily created the variable fool when it encountered it, and gave it the default value of nothing.

## Rules of variable names
1. Variable name may consist of alphanumeric characters and underscore characters.
1. The first character of variable name must be either a letter or an underscore.

## Make variable unchangeable
The shell makes no distinction between variables and constants; they are mostly for the programmer's convenience. A common convention is to use upper case letters to designate constants and lower case letters for true variables.

The shell actually does provide a way to enforce the immutability of constants, through with use of the `declare` builtin command with the `-r`(read-only) option:
```bash
$ declare -r FOO='yes'
$ FOO='no'
bash: FOO: readonly variable
```

## Assign value to variable
As we have seen, variables are assigned values this way:
```bash
variable=value
```
where variable is the name of variable and value is a string. Unlike some other programming languages, the shell does not care about the type of data assigned to a variable; it treats them all as strings.

The shell allows you to decalre a variable to have integer attribute, which guarantees that variable will always an integer value:
```bash
$ declare -i bar=-1
$ echo $bar
-1
$ bar='str'
$ echo $bar
0
```
As we can see, the string "str" is a non-integer value, so the value 0 is assigned instead.

Note that in an assignment, there must be no spaces between the variable name, the equals sign and the value. So what can the value consist of? Anything that we can expand into a string:
```bash
a=z                     # Assign the string "z" to variable a.
b="a string"            # Embedded spaces must be within quotes.
c="a string and $b"     # Other expansions such as variables can be expanded into the assignment.

d=$(ls -l foo.txt)      # Results of a command.
e=$((5 * 7))            # Arithmetic expansion.
f="\t\ta string\n"      # Escape sequences such as tabs and newlines.
```
During expansion, variable names may be surrounded by curly braces "{}". This is useful in cases where a variable name becomes ambiguous due to it s surrounding context. Here, we try to change the name of a file from 'myfile' to 'myfile1', using a variable:
```bash
$ filename='myfile'
$ touch $filename
$ mv $filename $filename1
mv: missing destination file operand after 'myfile'
```
This attempt fails because the shell interprets the second argument of the `mv` command as new (and empty) variable. The program can be overcome this way:
```bash
$ mv $filename ${filename}1
```
By adding the surrounding braces, the shell no longer interprets the trailing `1` as part of the variable name.                                                 

# Here Documents
There is a third way called a here document. A here document is an additional form of I/O redirection in which we embed a body of text into our script and feed it into the standarad input of a command. It works like this:
```bash
command << token
text
token
```
where `command` is the name of command is the name of command that accepts standdard input and `token` is a string used to indicate the end of the embedded text. Here we use `cat` and `_EOF_`  as an example:
```bash
$ foo = 'some text'
$ cat << _EOF_
> $foo
> "$foo"
> '$foo'
> \$foo
> _EOF_
some text
"some text"
'some text'
$foo
```
As we can see, the shell treats quotation marks as ordinary characters; this could turn out be handy at sometime.

If we change the redirection operator from `<<` to `<<-`, the shell will ignore the leading tab(not space) characters in the here document. This allows a here document to be indented, which can improve readability:
```bash
#!/bin/bash
cat <<- _EOF_
	1
        2
 	3
_EOF_
```

# Shell functions
Shell functions are "mini-scripts" that are located inside other scripts and can act as autonomous programs. Shell functions have two syntactic forms:
```bash
function name {
  commands
  return  
}

name () {
  commands
  return
}
``` 
where `name` is the name of function and `commands` are a series of commands contained within function. Shell function definitions must appear in the script before they are called.
## Local variables
A variable created without `local` qualifier is a global variable. Inside shell functions, it is often desirable to have local variables. Local variables are only accessible within the shell function in which they are defined and cease exist once the shell function terminates.

Here is an script that demonstrates how local variables are defined and used:
```bash
#!/bin/bash
foo=0 # global variable foo
funct_1 () {
  local foo # variable foo local to funct_1
  foo=1
  echo "funct_1: foo = $foo"
}

funct_2 () {
  local foo # variable foo local to funct_2
  foo=2
  echo "funct_2: foo = $foo"
}

echo "global: foo = $foo"
funct_1
echo "global: foo = $foo"
funct_2
echo "global: foo = $foo"
```
```bash
$ local_vars
global: foo = 0
funct_1: foo = 1
global: foo = 0
funct_2: foo = 2
global: foo = 0
```
## Shell functions in your .bashrc file
Shell functions make excellent replacements for `alias`, and are actually the preferred methond of creating small commands for personal use. Aliases are very limited in the kind of commands and shell features they support, whereas shell functions allow anything that can be scripted. For example, we could create a small fuction named `ds` for our .bashrc file:
```bash
ds () {
  echo "Disk Space Utilization for $HOSTNAME"
  df -h
}
```

# Flow Control
## Branching with if
The `if` statement has the following syntax:
```bash
if commands; then
  commands
[elif commands; then
  commands]
[else
  commands]
fi
```
where commands is a list of commands. For example:
```bash
#!/bin/bash
# script if_equal
x=5
if [ "$x" -eq 5]; then
  echo "x equals 5."
else
  echo "x does not equal 5."
fi
``` 
```bash
$ if_equal
x equals 5.
```
This is a little confusing at first glance. But before we can we can clear this up, we have to look at how the shell evaluates the success or failure of a command. 

## Exit status
Commands (including the scripts and shell functions we write) issue a value to the system when they terminate, called an exit status. This value, which is an integer in the range of 0 to 255, indicates the success or failure of the command's failure. The shell provides a parameter that we can use to examine the exit status. Here we see it in action:
```bash
$ ls -d /usr/bin
/usr/bin
$ echo $?
0
$ ls -d /bin/usr
ls: cannot access /bin/usr: No such file or directory
$ echo $?
2
```
The parameter `$?` records the exit the status values for the last command. Some commands use different exit status values to provide diagnostics for errors, while many commands simply exit with a value of 1 when they fail. Man pages often includes a section entitled "Exit Status", decribing what codes are used. However, a zero always indicates success.
