## User creation and password change from a CSV file 

```sh

# CSV file for creating users

ubuntu:~$ cat users.txt 
devuser1:Dev@123
devuser2:Dev@456
devuser3:Dev@789

ubuntu:~$ vi create-users.sh
ubuntu:~$ cat create-users.sh 

#!/bin/bash

FILE=users.txt

if [ ! -f $FILE ]; then
  echo "user file is not exist"
  exit 1
fi

while IFS=: read -r username password
do
  # Skip empty lines
  [[ -z "$username" || -z "$password" ]] && continue

  # Check if user already exists

  if id "$username" &>/dev/null; then
    echo "User $username already exists"
  else
    useradd -m "$username"
    echo "$username:$password" | chpasswd
    echo "User $username created successfully"
  fi
done < "$FILE"

ubuntu:~$ chmod +x create-users.sh 

ubuntu:~$ ./create-users.sh 
User devuser1 created successfully
User devuser2 created successfully
User devuser3 created successfully

# Verify user creation

ubuntu:~$ id devuser1
uid=1001(devuser1) gid=1001(devuser1) groups=1001(devuser1)
ubuntu:~$ id devuser2
uid=1002(devuser2) gid=1002(devuser2) groups=1002(devuser2)
ubuntu:~$ id devuser3
uid=1003(devuser3) gid=1003(devuser3) groups=1003(devuser3)
ubuntu:~$ 
