## Count total files in a directory
```sh
ubuntu:~$ cd /var/log
ubuntu:/var/log$ ls
README              apport.log  auth.log.1  cloud-init-output.log  dmesg       dmesg.2.gz  journal     killercoda  private   sysstat
alternatives.log    apt         btmp        cloud-init.log         dmesg.0     dpkg.log    kern.log    landscape   syslog    unattended-upgrades
alternatives.log.1  auth.log    btmp.1      dist-upgrade           dmesg.1.gz  dpkg.log.1  kern.log.1  lastlog     syslog.1  wtmp

ubuntu:/var/log$ cd
ubuntu:~$ vi count-files.md
ubuntu:~$ cat count-files.md 

#!/bin/bash

dir=$1
file_count=$(find $1 -type f | wc -l )

echo "Total file in $dir: $file_count"

ubuntu:~$ chmod +x count-files.md 

ubuntu:~$ ./count-files.md /var/log
Total file in /var/log: 37
ubuntu:~$ 
```
