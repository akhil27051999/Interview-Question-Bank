## CPU Load Average

**Purpose:**
  - Display the CPU load averages of the system.
  - Demonstrate command piping and text extraction using awk.
  - Useful for monitoring system load over different time intervals.

**Use Cases:**
  - Quick check for system performance.
  - Can be integrated into monitoring scripts to trigger alerts when load is high.
  - Useful in DevOps or SysAdmin automation for analyzing server health.

```sh
ubuntu:~$ vi cpu-load-average.sh
ubuntu:~$ cat cpu-load-average.sh 

#!/bin/bash

echo "CPU Load Average"
echo "----------------"

uptime | awk -F'load average:' '{print $2}'

ubuntu:~$ chmod +x cpu-load-average.sh 
ubuntu:~$ ./cpu-load-average.sh 
CPU Load Average
----------------
 0.05, 0.01, 0.00

ubuntu:~$ 
```

**Notes:**

- `-F` sets the field separator
- Here, the separator is the literal string: 
  ```text 
  load average
  ```
  
**awk splits the line into two parts:**
  
| Field | Content                               |
| ----- | ------------------------------------- |
| `$1`  | Everything **before** `load average:` |
| `$2`  | Everything **after** `load average:`  |
