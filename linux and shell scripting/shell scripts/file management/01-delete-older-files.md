
## Delete Files older then 30 days

**Purpose:**
  1. Identify and delete files older than a specified number of days.
  2. Demonstrates file timestamp checks, shell loops, and find command usage.
  3. Useful for cleaning up logs, temporary files, or old backups in automation scripts.

**Notes:**
  - `-mtime +30` → Matches files older than 30 days.
  - `-exec rm -f {} \;` → Deletes each matched file.
  - Using `ls -lh` before deletion is useful for logging and verification.
  - The script does not touch directories, only files.

**Use Cases:**
  - `Log rotation`: Remove old logs automatically.
  - `Temporary directories`: Clean up /tmp or cache directories.
  - `Backup maintenance`: Delete outdated backup files to save disk space.
  - `Cron automation`: Run the script daily/weekly to keep directories clean.

```sh

ubuntu:~$ mkdir /tmp/oldfiles
ubuntu:~$ touch /tmp/oldfiles/file{1..5}
ubuntu:~$ touch -d "40 days ago" /tmp/oldfiles/file1
ubuntu:~$ find /tmp/oldfiles -type f -mtime +30 -print
/tmp/oldfiles/file1

ubuntu:~$ cd /tmp/oldfiles/
ubuntu:/tmp/oldfiles$ ls -ltra
total 8
-rw-r--r--  1 root root    0 Dec  6 17:42 file1
drwxrwxrwt 16 root root 4096 Jan 15 17:42 ..
-rw-r--r--  1 root root    0 Jan 15 17:42 file5
-rw-r--r--  1 root root    0 Jan 15 17:42 file4
-rw-r--r--  1 root root    0 Jan 15 17:42 file3
-rw-r--r--  1 root root    0 Jan 15 17:42 file2
drwxr-xr-x  2 root root 4096 Jan 15 17:42 .

ubuntu:/tmp/oldfiles$ cd
ubuntu:~$ ./delete-old-files.sh /tmp/oldfiles/
-rw-r--r-- 1 root root 0 Dec  6 17:42 /tmp/oldfiles/file1

ubuntu:~$ cd /tmp/oldfiles/
ubuntu:/tmp/oldfiles$ ls -ltra
total 8
drwxrwxrwt 16 root root 4096 Jan 15 17:42 ..
-rw-r--r--  1 root root    0 Jan 15 17:42 file5
-rw-r--r--  1 root root    0 Jan 15 17:42 file4
-rw-r--r--  1 root root    0 Jan 15 17:42 file3
-rw-r--r--  1 root root    0 Jan 15 17:42 file2
drwxr-xr-x  2 root root 4096 Jan 15 17:49 .
ubuntu:/tmp/oldfiles$
```
