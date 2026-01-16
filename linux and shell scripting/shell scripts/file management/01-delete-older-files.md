
## Delete Files older then 20 days

**Purpose:**
  1. Identify and delete files older than a specified number of days.
  2. Demonstrates file timestamp checks, shell loops, and find command usage.
  3. Useful for cleaning up logs, temporary files, or old backups in automation scripts.

**Notes:**
  - `-mtime +20` → Matches files older than 20 days.
  - `-exec rm -f {} \;` → Deletes each matched file.
  - Using `ls -lh` before deletion is useful for logging and verification.
  - The script does not touch directories, only files.

**Use Cases:**
  - `Log rotation`: Remove old logs automatically.
  - `Temporary directories`: Clean up /tmp or cache directories.
  - `Backup maintenance`: Delete outdated backup files to save disk space.
  - `Cron automation`: Run the script daily/weekly to keep directories clean.

```sh

ubuntu:~$ touch -d "30 days ago" file{1..3}.txt
ubuntu:~$ ls
backup_status.txt  file1.txt  file2.txt  file3.txt  filesystem

ubuntu:~$ ls -ltrah
total 40K
-rw-r--r--  1 root root  161 Apr 22  2024 .profile
-rw-------  1 root root   10 Feb 10  2025 .bash_history
-rw-r--r--  1 root root    0 Dec 17 18:04 file3.txt
-rw-r--r--  1 root root    0 Dec 17 18:04 file2.txt
-rw-r--r--  1 root root    0 Dec 17 18:04 file1.txt
drwx------  2 root root 4.0K Dec 18 17:15 .ssh
drwxr-xr-x 22 root root 4.0K Dec 18 17:16 ..
lrwxrwxrwx  1 root root    1 Dec 18 17:16 filesystem -> /
-rw-r--r--  1 root root  109 Dec 18 17:16 .vimrc
-rw-r--r--  1 root root 3.2K Dec 18 17:16 .bashrc
-rw-r--r--  1 root root  165 Dec 18 17:16 .wget-hsts
drwxr-xr-x  4 root root 4.0K Jan 16 18:03 .theia
-rw-r--r--  1 root root   48 Jan 16 18:04 backup_status.txt
drwx------  4 root root 4.0K Jan 16 18:04 .

ubuntu:~$ find /* -name "*.txt" -type f -mtime +20 -print
/root/file1.txt
/root/file3.txt
/root/file2.txt

ubuntu:~$ vi remove-older-files.sh
ubuntu:~$ cat remove-older-files.sh 

#!/bin/bash

set -eou pipefail

dir=/root

echo "List out all the 20 days older files"
echo "------------------------------------"

find "$dir" -name "*.txt" -type f -mtime +20 -exec ls -lh {} \;

echo "Delete the files listed"
echo "-----------------------"

find "$dir" -name "*.txt" -type f -mtime +20 -exec rm -f {} \;

echo "Verify the filesystem post deletion"
echo "-----------------------------------"

find "$dir" -name "*.txt" -type f -mtime +20 -exec ls -lh {} \;

ubuntu:~$ chmod +x remove-older-files.sh 
ubuntu:~$ ./remove-older-files.sh

List out all the 30 days older files
------------------------------------
-rw-r--r-- 1 root root 0 Dec 17 18:04 /root/file1.txt
-rw-r--r-- 1 root root 0 Dec 17 18:04 /root/file3.txt
-rw-r--r-- 1 root root 0 Dec 17 18:04 /root/file2.txt

Delete the files listed
-----------------------

Verify the filesystem post deletion
-----------------------------------

ubuntu:~$ find /* -name "*.txt" -type f -mtime +20 -print 
ubuntu:~$

```
