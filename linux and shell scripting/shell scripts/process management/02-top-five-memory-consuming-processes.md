## Top Five Memory consuming processes

```sh
ubuntu:~$ vi process-monitor.sh
ubuntu:~$ cat process-monitor.sh

#!/bin/bash

echo "Top 5 memory consuming processes"
echo "--------------------------------"

ps -aux --sort=-%mem | head -n 6
ubuntu:~$ chmod +x process-monitor.sh 
ubuntu:~$ ./process-monitor.sh

Top 5 memory consuming processes
--------------------------------
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         928  0.0  3.6 1824352 70388 ?       Ssl  16:47   0:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root        1316  0.1  3.5 848960 68516 ?        SNl  16:47   0:01 /opt/theia/node /opt/theia/browser-app/src-gen/backend/main.js /root --hostname=0.0.0.0 --port 40205
root        1727  0.8  2.6 829224 52404 ?        SNl  16:54   0:01 /opt/theia/node /opt/theia/node_modules/@theia/core/lib/node/messaging/ipc-bootstrap --nsfwOptions={}
root         630  0.0  2.4 1654564 47612 ?       Ssl  16:47   0:00 /usr/bin/containerd
root         330  0.0  1.3 288952 27136 ?        SLsl 16:47   0:00 /sbin/multipathd -d -s

```
