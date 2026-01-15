
## Delete Files older then 30 days

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
