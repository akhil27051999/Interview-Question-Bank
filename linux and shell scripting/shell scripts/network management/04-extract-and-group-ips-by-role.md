## Extract & Group IPs by Role

**Purpose:**
  - Extract IP addresses from a structured inventory file (servers.txt)
  - Group and display IPs by role (nginx, nodejs, mysql, etc.)
  - Provide a clear, readable inventory view for operations and automation

**Use:**
  - `$1` → Accepts the server inventory file as input
  - `awk '{print $5}'` → Extracts roles
  - `sort -u` → Removes duplicate roles

**Use Cases:**
  - Service-wise server inventory
  - Preparing inputs for:
    - Ansible
    - SSH automation
    - Monitoring configuration
  - Quick infra audits
  - DevOps interview scripting tasks
  - Filtering production vs dev servers (can be extended)

**Tips:**
  - Blank lines are safely ignored because $5 won’t match
  - Sorting roles improves readability
  - Can be extended to:
    - Filter by environment (prod/dev)
    - Export to CSV / JSON
    - Accept role as a command-line argument

```sh

ubuntu:~$ vi servers.txt
ubuntu:~$ cat servers.txt   
web-01     192.168.10.11   ubuntu   prod   nginx
web-02     192.168.10.12   ubuntu   prod   nginx
app-01     192.168.20.21   ubuntu   prod   nodejs
app-02     192.168.20.22   ubuntu   prod   nodejs
db-01      192.168.30.31   ubuntu   prod   mysql
cache-01   192.168.40.41   ubuntu   prod   redis

web-dev-01 10.0.10.11      ubuntu   dev    nginx
app-dev-01 10.0.20.21      ubuntu   dev    nodejs
db-dev-01  10.0.30.31      ubuntu   dev    mysql

bastion-01 192.168.1.10    ubuntu   prod   ssh
monitor-01 192.168.50.50   ubuntu   prod   prometheus

ubuntu:~$ vi extracts-ips.sh 
ubuntu:~$ cat extracts-ips.sh

#!/bin/bash

if [ ! -f "$1" ]; then
  echo "file is not found"
  exit 1
fi

echo "Extracting IP from $1"
echo "------------------------------"

awk '{print $2}' "$1"

ubuntu:~$ ./extracts-ips.sh server.txt
file is not found
ubuntu:~$ ./extracts-ips.sh servers.txt

Extracting IP from servers.txt
------------------------------
cat
192.168.10.11
192.168.10.12
192.168.20.21
192.168.20.22
192.168.30.31
192.168.40.41

10.0.10.11
10.0.20.21
10.0.30.31

192.168.1.10
192.168.50.50

ubuntu:~$ vi extracts-ips.sh
ubuntu:~$ cat extracts-ips.sh 

#!/bin/bash

if [ ! -f "$1" ]; then
  echo "file is not found"
  exit 1
fi

echo "Extracting IP from $1"
echo "------------------------------"

roles=$(awk '{print $5}' "$1" | sort -u)

for role in $roles; do
  echo "Role: $role"
  awk -v r="$role" '$5 == r {print "  " $2}' "$1"
done

ubuntu:~$ ./extracts-ips.sh servers.txt
Extracting IP from servers.txt
------------------------------
Role: mysql
  192.168.30.31
  10.0.30.31
Role: nginx
  192.168.10.11
  192.168.10.12
  10.0.10.11
Role: nodejs
  192.168.20.21
  192.168.20.22
  10.0.20.21
Role: prometheus
  192.168.50.50
Role: redis
  192.168.40.41
Role: ssh
  192.168.1.10

```
