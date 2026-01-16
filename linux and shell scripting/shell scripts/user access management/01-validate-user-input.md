## Validate User Input 

**Purpose:**
  1. Accept user input at runtime using the terminal.
  2. Validate whether the entered value is positive, negative, or zero.
  3. Demonstrate conditional logic (if / elif / else) in shell scripting.

**use:**
  - `read -p` → Prompts the user and stores input in a variable (num).
  - `-gt` → Checks if the number is greater than zero.
  - `-lt` → Checks if the number is less than zero.
  - `if / elif / else` → Executes logic based on the condition outcome.
  - `echo` → Displays the result to the user.

**Use Cases:**
  - Validating user-provided values in shell scripts.
  - Simple input checks before running critical operations.
  - Used in menu-driven scripts or automation tools.
  - Basic validation for DevOps utility scripts (ports, thresholds, IDs).
    
```sh
ubuntu:~$ vi user-script.sh
ubuntu:~$ chmod +x user-script.sh 
ubuntu:~$ cat user-script.sh 

#!/bin/bash

read -p "enter the value:" num

if [ "$num" -gt 0 ]; then 
  echo "Its a positive number"
elif [ "$num" -lt 0 ]; then
  echo "Its a negitive number"
else
  echo "Its a zero"
fi

ubuntu:~$ ./user-script.sh 
enter the value:20
Its a positive number
ubuntu:~$ ./user-script.sh 
enter the value:-19
Its a negitive number
ubuntu:~$ ./user-script.sh 
enter the value:0
Its a zero
```
