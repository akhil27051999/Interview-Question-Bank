## Check whether a username exist or not in the system

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
