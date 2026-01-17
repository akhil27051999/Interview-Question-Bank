## Cron Job Troubleshooting

**Purpose:**
  1. Inspected a broken cron job
  2. Diagnosed why it wasn't producing output
  3. Fixed the cron job
  4. Verified it runs and writes visible output to a file

```sh
--------------- # Step 1: Check Existing Cron jobs ---------------

ubuntu:~$ crontab -l
* * * * * /usr/local/bin/backup.sh

ubuntu:~$ ls
backup_status.txt  filesystem

# → The problem is that the script do not print anything in the file ~/backup_status.txt .

--------------- # Step 2: Inspect Cron logs ---------------

ubuntu:~$ cat backup_status.txt 
Sat Jan 17 09:36:01 UTC 2026 - Backup completed
Sat Jan 17 09:37:01 UTC 2026 - Backup completed

# → Cron jobs run in a minimal environment and may need full paths or output redirection to work correctly.

--------------- # Step 3: Fix broken Cron jobs ---------------

ubuntu:~$ crontab -e
Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 1
crontab: installing new crontab

ubuntu:~$ crontab -l
* * * * * /usr/local/bin/backup.sh >> ~/backup_status.txt 2>&1

# → This ensures the output is appended to the visible backup_status.txt file.

--------------- # Step 4: Verify the Cron Jobs ---------------

ubuntu:~$ cat backup_status.txt 
Sat Jan 17 09:36:01 UTC 2026 - Backup completed
Sat Jan 17 09:37:01 UTC 2026 - Backup completed
Sat Jan 17 09:38:01 UTC 2026 - Backup completed
Sat Jan 17 09:39:01 UTC 2026 - Backup completed
Sat Jan 17 09:40:01 UTC 2026 - Backup completed
Sat Jan 17 09:41:01 UTC 2026 - Backup completed

# → The cron job is now running correctly, and you can visibly see the results.

ubuntu:~$ 
```
