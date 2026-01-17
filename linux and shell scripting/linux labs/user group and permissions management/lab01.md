## User Group and Permission Troubleshooting

### Step 1: Create Users and Groups
  1. Create a group called devteam
  2. Create two users and add them to the group
  3. Set passwords

```sh
ubuntu:~$ sudo groupadd devteam

ubuntu:~$ sudo useradd -m -s /bin/bash akhil
ubuntu:~$ sudo useradd -m -s /bin/bash atchyuth

ubuntu:~$ sudo usermod -aG devteam akhil
ubuntu:~$ sudo usermod -aG devteam atchyuth

ubuntu:~$ echo "akhil:Akhil@1999" | sudo chpasswd
ubuntu:~$ echo "atchyuth:Abhi@7777" | sudo chpasswd
```
> Now you have two users (akhil and atchyuth ) both belonging to the devteam group.

### Step 2: Set File Permissions
  1. Create a project folder and file
  2. Change ownership to the devteam group
  3. Set permissions so that only the owner can read/write
     
```sh
ubuntu:~$ sudo mkdir -p /srv/project
ubuntu:~$ echo "Top Secret Project File" | sudo tee /srv/project/data.txt
Top Secret Project File

ubuntu:~$ cd /srv/project/
ubuntu:/srv/project$ ls -lh data.txt 
-rw-r--r-- 1 root root 24 Jan 17 14:47 data.txt
ubuntu:/srv/project$ sudo chown :devteam data.txt 
ubuntu:/srv/project$ ls -lh data.txt 
-rw-r--r-- 1 root devteam 24 Jan 17 14:47 data.txt
ubuntu:/srv/project$ sudo chmod 600 data.txt          
ubuntu:/srv/project$ ls -lh data.txt 
-rw------- 1 root devteam 24 Jan 17 14:47 data.txt
```
> At this point, akhil and atchyuth should NOT be able to read the file, even though they’re in the devteam group. This sets up our troubleshooting.

### Step 3: Test Access
  1. Switch to alice and try reading the file
```sh
ubuntu:/srv/project$ su - akhil
akhil@ubuntu:~$ cat /srv/project/data.txt 
cat: /srv/project/data.txt: Permission denied
```
> You should see a “Permission denied” error.
> This is expected, since the file’s mode is 600 and doesn’t allow group access.

### Step 4: Debug and Fix Permissions
  1. Exit back to root
  2. Change file permissions so the group can read
  3. Verify permissions
     
```sh
akhil@ubuntu:~$ exit
logout
ubuntu:/srv/project$ sudo chmod 640 /srv/project/data.txt 
ubuntu:/srv/project$ ls -lh data.txt 
-rw-r----- 1 root devteam 24 Jan 17 14:47 data.txt
```
> Now the group has read access.

### Step 5: Verify Access
  1. Switch back to alice and test
     
```sh
ubuntu:/srv/project$ su - akhil
akhil@ubuntu:~$ cat /srv/project/data.txt 
Top Secret Project File
akhil@ubuntu:~$
```
