## User Creation and Password Change from a CSV File 

**Purpose:**
  1. Read a CSV-style file (users.txt) containing username:password pairs.
  2. Create Linux users with the specified usernames and passwords.
  3. Demonstrate file handling, loops, conditional checks, and password setting in shell scripting.
  4. Ensure idempotency: does not create users if they already exist.

**Use Cases:**
  - Batch user creation on Linux servers from a CSV file.
  - Useful in DevOps / SysAdmin automation, e.g., provisioning accounts for teams.
  - Can be integrated into cloud instance setup scripts or CI/CD pipelines.
  - Prevents errors by checking for existing users before creation.

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
