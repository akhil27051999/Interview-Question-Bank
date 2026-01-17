## Linux Systemd Service Debugging

### Step 1: Create a Custom Service
- Let’s start by creating a new systemd service unit.
  
** Create a script that prints a timestamp every 5 seconds:**
```sh
ubuntu:~$ mkdir -p /opt/myapp
ubuntu:~$ echo -e '#!/bin/bash\nwhile true; do echo "MyApp running: $(date)"; sleep 5; done' > /opt/myapp/myapp.sh
ubuntu:~$ chmod +x /opt/myapp/myapp.sh
```

### Create a systemd service unit file:
```sh
ubuntu:~$ cat <<EOF | sudo tee /etc/systemd/system/myapp.service
[Unit]
Description=MyApp Custom Service
After=network.target

[Service]
ExecStart=/opt/myapp/myappp.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
>  Notice that we’ve intentionally introduced a typo: myappp.sh (three p’s). This will cause the service to fail when started, and you’ll debug it later.

### Step 2: Enable and Start the Service

**Now let’s try to start the service.**

1. Reload systemd so it picks up the new service:
```sh
sudo systemctl daemon-reload
```
2. Enable the service to start at boot:
```sh
sudo systemctl enable myapp.service
```
3. Start the service:
```sh
sudo systemctl start myapp.service
```
4. Check the service status:
```sh
systemctl status myapp.service
```
> You should see that the service fails to start. Don’t worry — this is expected! You’ll debug it in the next step.

```sh
ubuntu:~$ sudo systemctl daemon-reload
ubuntu:~$ sudo systemctl enable myapp.service
Created symlink /etc/systemd/system/multi-user.target.wants/myapp.service → /etc/systemd/system/myapp.service.

ubuntu:~$ sudo systemctl start myapp.service
ubuntu:~$ systemctl status myapp.service
× myapp.service - MyApp Custom Service
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Thu 2026-01-15 11:10:05 UTC; 30s ago
   Duration: 296us
    Process: 1873 ExecStart=/opt/myapp/myappp.sh (code=exited, status=203/EXEC)
   Main PID: 1873 (code=exited, status=203/EXEC)
        CPU: 737us

Jan 15 11:10:05 ubuntu systemd[1]: Started myapp.service - MyApp Custom Service.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Main process exited, code=exited, status=203/EXEC
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Failed with result 'exit-code'.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Scheduled restart job, restart counter is at 5.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Start request repeated too quickly.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Failed with result 'exit-code'.
Jan 15 11:10:05 ubuntu systemd[1]: Failed to start myapp.service - MyApp Custom Service.
```

### Step 3: Debug Service Failures

**Let’s investigate why the service failed.**

1. Check the status again:
```sh
systemctl status myapp.service
```
2. Use the logs:
```sh
journalctl -xeu myapp.service
```

> You should see an error message indicating that the executable file could not be found: /opt/myapp/myappp.sh: No such file or directory .
> This means our ExecStart command is wrong.

```sh
ubuntu:~$ journalctl -xeu myapp.service
░░ Support: http://www.ubuntu.com/support
░░ 
░░ Automatic restarting of the unit myapp.service has been scheduled, as the result for
░░ the configured Restart= setting for the unit.
Jan 15 11:10:05 ubuntu systemd[1]: Started myapp.service - MyApp Custom Service.
░░ Subject: A start job for unit myapp.service has finished successfully
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░ 
░░ A start job for unit myapp.service has finished successfully.
░░ 
░░ The job identifier is 2662.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Main process exited, code=exited, status=203/EXEC
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░ 
░░ An ExecStart= process belonging to unit myapp.service has exited.
░░ 
░░ The process' exit code is 'exited` and its exit status is 203.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░ 
░░ The unit myapp.service has entered the 'failed' state with result 'exit-code'.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Scheduled restart job, restart counter is at 5.
░░ Subject: Automatic restarting of a unit has been scheduled
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░ 
░░ Automatic restarting of the unit myapp.service has been scheduled, as the result for
░░ the configured Restart= setting for the unit.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Start request repeated too quickly.
Jan 15 11:10:05 ubuntu systemd[1]: myapp.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░ 
░░ The unit myapp.service has entered the 'failed' state with result 'exit-code'.
Jan 15 11:10:05 ubuntu systemd[1]: Failed to start myapp.service - MyApp Custom Service.
░░ Subject: A start job for unit myapp.service has failed
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░ 
░░ A start job for unit myapp.service has finished with a failure.
░░ 
░░ The job identifier is 2776 and the job result is failed.
```

### Step 4: Fix the Service

**Now let’s fix the error.**
1. Edit the unit file:
```sh
sudo nano /etc/systemd/system/myapp.service
```
2. Correct the line:
```sh
ExecStart=/opt/myapp/myapp.sh
```
3. Reload systemd:
```sh
sudo systemctl daemon-reload
```
4. Restart the service:
```sh
sudo systemctl restart myapp.service
```
5. Check status again:
```sh
systemctl status myapp.service
```
Now the service should be running properly.

```sh
ubuntu:~$ sudo nano /etc/systemd/system/myapp.service
ubuntu:~$ sudo systemctl daemon-reload

ubuntu:~$ sudo systemctl restart myapp-service
Failed to restart myapp-service.service: Unit myapp-service.service not found.

ubuntu:~$ sudo systemctl restart myapp.service
ubuntu:~$ systemctl status myapp.service

● myapp.service - MyApp Custom Service
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-01-15 11:13:30 UTC; 10s ago
   Main PID: 1939 (myapp.sh)
      Tasks: 2 (limit: 2237)
     Memory: 584.0K (peak: 832.0K)
        CPU: 8ms
     CGroup: /system.slice/myapp.service
             ├─1939 /bin/bash /opt/myapp/myapp.sh
             └─1945 sleep 5

Jan 15 11:13:30 ubuntu systemd[1]: Started myapp.service - MyApp Custom Service.
Jan 15 11:13:30 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:30 UTC 2026
Jan 15 11:13:35 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:35 UTC 2026
Jan 15 11:13:40 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:40 UTC 2026
```

### Step 5: Verify and Test

**Finally, let’s confirm everything is working.**

1. Check that the process is running:
```sh
ps aux | grep myapp
```
2. View logs:
```sh
journalctl -u myapp.service -f
```
3. You should see output like:
```
MyApp running: Mon Sep 16 14:32:21 UTC 2025
MyApp running: Mon Sep 16 14:32:26 UTC 2025
```

```sh
ubuntu:~$ ps aux | grep myapp 
root        1939  0.0  0.1   7740  3712 ?        Ss   11:13   0:00 /bin/bash /opt/myapp/myapp.sh
root        1955  0.0  0.0   3528  1792 pts/0    S+   11:13   0:00 grep --color=auto myapp

ubuntu:~$ journalctl -u myapp.service -f
Jan 15 11:13:30 ubuntu systemd[1]: Started myapp.service - MyApp Custom Service.
Jan 15 11:13:30 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:30 UTC 2026
Jan 15 11:13:35 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:35 UTC 2026
Jan 15 11:13:40 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:40 UTC 2026
Jan 15 11:13:45 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:45 UTC 2026
Jan 15 11:13:50 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:50 UTC 2026
Jan 15 11:13:55 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:13:55 UTC 2026
Jan 15 11:14:00 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:00 UTC 2026
Jan 15 11:14:05 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:05 UTC 2026
Jan 15 11:14:10 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:10 UTC 2026
Jan 15 11:14:15 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:15 UTC 2026
Jan 15 11:14:20 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:20 UTC 2026
Jan 15 11:14:25 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:25 UTC 2026
Jan 15 11:14:30 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:30 UTC 2026
Jan 15 11:14:35 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:35 UTC 2026
Jan 15 11:14:40 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:40 UTC 2026
Jan 15 11:14:45 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:45 UTC 2026
Jan 15 11:14:50 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:50 UTC 2026
Jan 15 11:14:55 ubuntu myapp.sh[1939]: MyApp running: Thu Jan 15 11:14:55 UTC 2026
```
