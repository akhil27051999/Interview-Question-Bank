## Count servers by role using sort and uniq

**Purpose:**
  1. Count the number of servers grouped by their role/application (like nginx, nodejs, mysql).
  2. Identify which roles are most common in your environment.

**Use:**
  - `awk` → To extract the column with the server role/application.
  - `grep -v '^$'` → Remove empty lines for accurate counting.
  - `sort` → Alphabetically sort before using uniq.
  - `uniq -c` → Count occurrences of each unique role.
  - `sort -nr` → Sort by frequency, most common first.

**Use Cases:**
  - Quickly see how many servers run each application/service.
  - Generate inventory or reporting for Ops/DevOps teams.
  - Identify over-provisioned or under-utilized services.
  - Can be extended to group by environment (prod/dev) or region by changing the awk column.

**Tips:**
  - Always sort alphabetically before uniq -c so counts are correct.
  - Use sort -nr to see most frequent roles first.
  - Can combine with awk '{print $4, $5}' to count by environment + role.
    
```sh
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

ubuntu:~$ vi count-servers.sh
ubuntu:~$ chmod +x count-servers.sh   
ubuntu:~$ cat count-servers.sh 
#!/bin/bash

awk '{print $5}' servers.txt | grep -v '^$' | sort | uniq -c

ubuntu:~$ ./count-servers.sh 
      2 mysql
      3 nginx
      3 nodejs
      1 prometheus
      1 redis
      1 ssh

ubuntu:~$ 
# Always sort alphabetically first before uniq -c
# Use sort -nr after counting to sort by frequency
# Don’t use -n on strings

ubuntu:~$ vi count-servers.sh 
ubuntu:~$ cat count-servers.sh 
#!/bin/bash

awk '{print $5}' servers.txt | grep -v '^$' | sort | uniq -c | sort -nr

ubuntu:~$ ./count-servers.sh 
      3 nodejs
      3 nginx
      2 mysql
      1 ssh
      1 redis
      1 prometheus

# grep -v '^$'→ remove empty spaces
# sort → alphabetically
# uniq -c → count occurrences
# sort -nr → sort numerically descending by count
```
