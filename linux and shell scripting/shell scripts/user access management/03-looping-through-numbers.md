## Looping Through Numbers 1 → 20

**Purpose:**
  1. Iterate through numbers 1 to 20 using a for loop.
  2. Demonstrate conditional logic (if / else) in shell scripting.
  3. Classify each number as “positive” or “negative” (based on even/odd in this script).
  4. Practice arithmetic operations in Bash (% modulo operator).

**Use Cases:**
  - Learning loops and conditional statements in shell scripting.
  - Classifying numbers or performing batch arithmetic operations.
  - Can be extended for menu-driven scripts or data validation.
    
```sh
ubuntu:~$ vi integer-classification.sh
ubuntu:~$ cat integer-classification.sh 

#!/bin/bash

for i in {1..20}; do
  if (( i%2 == 0 )); then
    echo "$i is a positive number"
  else
    echo "$i is a negative number"
  fi
done 

ubuntu:~$ chmod +x integer-classification.sh 
ubuntu:~$ ./integer-classification.sh 
1 is a negative number
2 is a positive number
3 is a negative number
4 is a positive number
5 is a negative number
6 is a positive number
7 is a negative number
8 is a positive number
9 is a negative number
10 is a positive number
11 is a negative number
12 is a positive number
13 is a negative number
14 is a positive number
15 is a negative number
16 is a positive number
17 is a negative number
18 is a positive number
19 is a negative number
20 is a positive number
```
