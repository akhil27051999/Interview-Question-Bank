## Validate IP address Format

**Purpose:**
  1. Validate whether a given IPv4 address is in a correct format.
  2. Demonstrates user input handling, regex matching, and conditional statements in Bash.
  3. Useful for network configuration validation or input checks in scripts.

**Use Cases:**
  - `Network scripts`: Verify user-provided IPs before assigning to interfaces.
  - `Automation tools`: Check IP inputs in server provisioning scripts.
  - `Validation in DevOps pipelines`: Ensure correct configuration before deployment.
    
```sh
ubuntu:~$ vi validate-ip.sh
ubuntu:~$ cat validate-ip.sh 

#!/bin/bash

echo "Validate IP address format"
echo "----------------------------"

read -p "Enter the IP address: " ip

if [[ $ip =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
  echo "Valid IP address format"
else
  echo "Invalid IP address format"
fi

ubuntu:~$ chmod +x validate-ip.sh 
ubuntu:~$ ./validate-ip.sh 
Validate IP address format
----------------------------
Enter the IP address: 455.455.4.4
Valid IP address format

ubuntu:~$ ./validate-ip.sh 
Validate IP address format
----------------------------
Enter the IP address: 255.255.255
Invalid IP address format

ubuntu:~$ 
```
