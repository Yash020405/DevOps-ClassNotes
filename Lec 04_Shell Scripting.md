# Shell Scripting – Class Notes

## Introduction to Shell Scripting

- Shell scripting is used to automate tasks in UNIX and Linux environments.
- A shell script is a simple program executed by a shell, which acts as a command-line interpreter.
- Commonly used shells discussed were `/bin/sh` and `/bin/bash`.

---

## Creating and Executing Shell Scripts

- Shell scripts are created with a `.sh` extension.
- Every script starts with a **shebang** that specifies the interpreter.
```bash
#!/bin/bash
```
- Scripts are made executable using:
```bash
chmod +x script.sh
```
- Scripts are executed using:
```bash
./script.sh
```

---

## Shebang and Interpreters

- The shebang (`#!`) tells the system which interpreter should execute the script.
- Interpreters discussed:
- `/bin/bash` – Feature-rich shell
- `/bin/sh` – POSIX-compliant and lightweight
- `/usr/bin/env bash` – Finds bash from the user’s PATH
- Other interpreters such as Python and Perl can also be specified

---

## Variables and User Input

- Variables are defined using:
```bash
VARIABLE=value
# Example:
USERNAME="JohnDoe"
```
- By convention, variable names are written in uppercase.
- Variables are accessed using:
```bash
$VARIABLE
```
- User input is taken using:
```bash
read -p "Prompt: " VARIABLE
```

---

## Conditionals and Loops

- Conditional execution is done using `if` statements.
- Conditions are evaluated using single brackets `[ ]`.
- File existence is checked using:
- `-f` for files
- Loops discussed include:
- `for` loop
- `while` loop

---

## File and Directory Management

- Commonly used commands:
- `ls`, `pwd`, `cd`
- `mkdir` to create directories
- `touch` or `cat > filename` to create files
- `rm`, `cp`, `mv`
- The command `rm -rf` removes directories recursively and forcefully and should be used with caution.

---

## File Permissions

- Permissions are defined for:
- User
- Group
- Others
- Permission types:
- `r` – read
- `w` – write
- `x` – execute
- Permissions are modified using `chmod`.

---

## Access Control Lists (ACLs)

- ACLs provide fine-grained permission control beyond the traditional user, group, and others model.
- ACLs allow multiple users and groups to have different permissions on the same file or directory.
- ACLs can be viewed using:
```bash
getfacl filename
```
- ACLs can be set or modified using:
```bash
setfacl
```
- ACL entries can be removed or reset when required.
- Default ACLs can be applied so that newly created files inherit permissions.

---

## String Manipulation

- Text processing commands discussed:
- `tr` – translate or delete characters
- `sed` – stream editor for text transformation
- `awk` – pattern scanning and processing
- Uppercasing can be achieved with:
```bash
echo $STRING | tr [a-z] [A-Z]
```
- String length can be obtained using parameter expansion.
- Bash supports advanced parameter expansion for:
- Uppercase conversion
- Lowercase conversion
- Substring extraction
- These features are Bash-specific and do not work in `/bin/sh`.

---

## Find and Locate Commands

- The `find` command is powerful for searching files based on patterns, types, and other attributes.
- Syntax:
```bash
find <path> -name <pattern>
```
- Can search by file type, size, modification time, and permissions.
- Regular expressions (regex) are vital for text processing within the shell.

---

## Difference Between /bin/sh and /bin/bash

- `/bin/sh` is lightweight, POSIX-compliant, and highly portable.
- `/bin/bash` provides advanced features such as:
- Arrays
- Enhanced conditionals (`[[ ]]`)
- Advanced string manipulation
- Scripts using Bash-specific features must be executed with Bash and not with `sh`.

---

## Functions in Shell Scripts

- Functions are used to group reusable logic.
- A function is defined using:
```bash
function_name() {
    commands
}
```
- Functions are called using their name.

---

## User and Group Management with Scripts

- Shell scripts can automate:
- Group creation using `groupadd`
- User creation using `useradd`
- Shell assignment (setting default shells)
- Home directory setup
- Welcome messages and permissions
- Proper scripts include:
- Root user checks
- Argument validation
- Existence checks for users and groups

---

## Reusable Scripts and Sourcing

- Scripts can be split into:
- Utility scripts
- Main scripts
- Utility scripts are sourced using:
```bash
source utils.sh
```
- This allows functions to be reused across multiple scripts.