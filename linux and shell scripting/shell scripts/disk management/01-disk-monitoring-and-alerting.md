## Disk Monitoring Script

**Purpose:**
  1. check the total disk consumption of the system
  2. setup threshold and alert if disk exceeds the threshold
   
**Use:**
 - `awk` → To find patterns and perform specified actions on matching lines or data fields
 - `sed` → Use for searching, replacing, inserting, and deleting text without opening the file interactively

**Use Cases:**
  - Monitor disk usage on servers to prevent full disks and system issues.
  - Automate alerts for system administrators.
  - Can be scheduled via cron to run periodically and report disk status.

**Tips / Notes:**
  - Always use `df -h` (not `du -sh`) for filesystem usage, because du shows folder sizes, not total disk usage.
  - Ensure numeric comparison uses `[ "$usage" -ge "$threshold" ]` (remove extra than).
  - Can extend the script to email alerts when usage exceeds the threshold.
  - For multiple partitions, loop through `df -h` lines instead of just `NR==2`.
    
```sh

ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  976K  190M   1% /run
/dev/vda1        19G  5.2G   14G  29% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi

ubuntu:~$ vi monitor-disk-usage.sh
ubuntu:~$ cat monitor-disk-usage.sh

#!/bin/bash

threshold=20

usage=$(df -h /* | awk 'NR==2 {print $5}' | sed 's/%/ /');

if [ "$usage -ge than $threshold" ]; then
  echo "disk usage is higher than $threshold : $usage"
else
  echo "disk usage is normal"
fi

ubuntu:~$ ./monitor-disk-usage.sh 
disk usage is higher than 20 : 29 
```
