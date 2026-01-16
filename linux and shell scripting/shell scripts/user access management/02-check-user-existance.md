## Check whether a username exist or not in the system

**Purpose:**
  1. Accept a username input from the user at runtime using the terminal.
  2. Check whether the entered username exists on the system.

**Use Cases:**
  - Validating user input in shell scripts.
  - Checking before creating or deleting users to avoid errors.

```sh

ubuntu:~$ id
uid=0(root) gid=0(root) groups=0(root)

ubuntu:~$ vi user-exists.sh
ubuntu:~$ cat user-exists.sh 

#!/bin/bash

read -p "enter the username: " username

if id $username 2&>/dev/null; then
  echo "$username exist"
else
  echo "$username doesn't exist"
fi

ubuntu:~$ chmod +x user-exists.sh

ubuntu:~$ ./user-exists.sh root
enter the username: root
root exist

ubuntu:~$ ./user-exists.sh     
enter the username: akhil
akhil doesn't exist

```
