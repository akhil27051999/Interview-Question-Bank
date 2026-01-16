## CPU Load Average

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
- awk splits the line into two parts:
| Field | Content                               |
| ----- | ------------------------------------- |
| `$1`  | Everything **before** `load average:` |
| `$2`  | Everything **after** `load average:`  |
