## Text Transformation with sed

**purpose:**
  1. Create a multi-line file quickly using a here-document.
  2. Perform basic text substitution using the s/ command.
  3. Delete lines based on a pattern.
  4. Make changes permanent using the -i (in-place) flag.

### Step 1: Create the Input File
**We need a configuration file to practice with. We will use a here-document (<< EOF...EOF ) piped into cat to create the multi-line file app.conf in a single command.**

```sh
ubuntu:~$ cat > app.conf << EOF
# Application Configuration
HOST=127.0.0.1
PORT=80
TIMEOUT=30
DEBUG=false
# END
EOF

ubuntu:~$ cat app.conf 
# Application Configuration
HOST=127.0.0.1
PORT=80
TIMEOUT=30
DEBUG=false
# END
```

### Step 2: Substitution with sed
**The most common use of sed is to replace text using the substitution command: s/pattern/replacement/ .**
>  `Important Note`: By default, sed prints the result to your terminal screen (stdout) and does not change the original file.

**Observe the File**
> Confirm that the original file app.conf is still unchanged (it should still show PORT=80):
```sh
cat app.conf
```

```sh
ubuntu:~$ sed 's/PORT=80/PORT=8080/' app.conf
# Application Configuration
HOST=127.0.0.1
PORT=8080
TIMEOUT=30
DEBUG=false
# END

ubuntu:~$ cat app.conf 
# Application Configuration
HOST=127.0.0.1
PORT=80
TIMEOUT=30
DEBUG=false
# END
```

### Step 3: Line Deletion
**You can use the d command in sed to delete lines that match a specific pattern.**
  - Delete all comment lines (lines starting with # ).
> We use the pattern address /^#/ which means "start of line (^ ) followed by a hash (# )", combined with the delete command (d ).

```sh
ubuntu:~$ sed '/^#/d' app.conf
HOST=127.0.0.1
PORT=80
TIMEOUT=30
DEBUG=false

ubuntu:~$ cat app.conf 
# Application Configuration
HOST=127.0.0.1
PORT=80
TIMEOUT=30
DEBUG=false
# END
```

### Step 4: In-Place Editing
**To make the changes permanent and save them back to the original file, you must use the -i flag (for in-place editing).**
  - Permanently change DEBUG=false to DEBUG=true in app.conf

**Verification:**
  - Use cat one last time. If the file now permanently contains DEBUG=true, you have successfully completed the core task!
    
```sh
ubuntu:~$ sed -i 's/DEBUG=false/DEBUG=true/' app.conf

ubuntu:~$ cat app.conf 
# Application Configuration
HOST=127.0.0.1
PORT=80
TIMEOUT=30
DEBUG=true
# END
```
