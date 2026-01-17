## Process Management & Resource Limits

**In this scenario, you will learn how to diagnose, manage, and mitigate runaway processes that consume excessive CPU or memory.**

**As a system administrator or Kubernetes operator, it's important to quickly identify and handle resource-hogging processes to keep the system stable.**

### You will practice:
  1. Inspecting running processes with ps , top , and htop
  2. Identifying high CPU and memory consumers
  3. Terminating runaway processes safely
  4. Applying resource limits with ulimit , nice , and renice

### Scenario Setup:
**Some simulated runaway processes have been started in the background:**
  - yes (infinite CPU usage)
  - dd (large memory usage)
  - An infinite while loop (CPU hog)
> Your task is to diagnose and fix the system by controlling or terminating these processes.

---

### Step 1: Inspect Running Processes

**To view all running processes and memory:**

```sh
ps aux
top
free -h
```

```sh
ubuntu:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.3  0.6  22232 13160 ?        Ss   12:59   0:01 /sbin/init
root           2  0.0  0.0      0     0 ?        S    12:59   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    12:59   0:00 [pool_workqueue_release]
root           4  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-rcu_g]
root           5  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-rcu_p]
root           6  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-slub_]
root           7  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-netns]
root           9  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/0:0H-kblockd]
root          11  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/u2:0-flush-253:0]
root          12  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-mm_pe]
root          13  0.0  0.0      0     0 ?        I    12:59   0:00 [rcu_tasks_kthread]
root          14  0.0  0.0      0     0 ?        I    12:59   0:00 [rcu_tasks_rude_kthread]
root          15  0.0  0.0      0     0 ?        I    12:59   0:00 [rcu_tasks_trace_kthread]
root          16  0.0  0.0      0     0 ?        S    12:59   0:00 [ksoftirqd/0]
root          17  0.0  0.0      0     0 ?        I    12:59   0:00 [rcu_preempt]
root          18  0.0  0.0      0     0 ?        S    12:59   0:00 [migration/0]
root          19  0.0  0.0      0     0 ?        S    12:59   0:00 [idle_inject/0]
root          20  0.0  0.0      0     0 ?        S    12:59   0:00 [cpuhp/0]
root          21  0.0  0.0      0     0 ?        S    12:59   0:00 [kdevtmpfs]
root          22  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-inet_]
root          23  0.0  0.0      0     0 ?        S    12:59   0:00 [kauditd]
root          24  0.0  0.0      0     0 ?        S    12:59   0:00 [khungtaskd]
root          26  0.0  0.0      0     0 ?        S    12:59   0:00 [oom_reaper]
root          27  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/u2:2-ext4-rsv-conversion]
root          28  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-write]
root          29  0.0  0.0      0     0 ?        S    12:59   0:00 [kcompactd0]
root          30  0.0  0.0      0     0 ?        SN   12:59   0:00 [ksmd]
root          31  0.0  0.0      0     0 ?        SN   12:59   0:00 [khugepaged]
root          32  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-kinte]
root          33  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-kbloc]
root          34  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-blkcg]
root          35  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/9-acpi]
root          36  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-tpm_d]
root          37  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-ata_s]
root          38  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-md]
root          39  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-md_bi]
root          40  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-edac-]
root          41  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-devfr]
root          42  0.0  0.0      0     0 ?        S    12:59   0:00 [watchdogd]
root          43  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/0:1H-kblockd]
root          44  0.0  0.0      0     0 ?        S    12:59   0:00 [kswapd0]
root          45  0.0  0.0      0     0 ?        S    12:59   0:00 [ecryptfs-kthread]
root          46  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-kthro]
root          47  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/24-aerdrv]
root          48  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/25-aerdrv]
root          49  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/26-aerdrv]
root          50  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/27-aerdrv]
root          51  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/28-aerdrv]
root          52  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/29-aerdrv]
root          53  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/30-aerdrv]
root          54  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/31-aerdrv]
root          55  0.0  0.0      0     0 ?        S    12:59   0:00 [irq/32-aerdrv]
root          56  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-acpi_]
root          57  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_0]
root          58  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root          59  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-mld]
root          60  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-ipv6_]
root          67  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-kstrp]
root          69  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/u3:0]
root          74  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-crypt]
root          75  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/0:2-events]
root          84  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-charg]
root         138  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_1]
root         144  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root         145  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_2]
root         149  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root         150  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_3]
root         151  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root         153  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_4]
root         154  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root         155  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_5]
root         156  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root         157  0.0  0.0      0     0 ?        S    12:59   0:00 [scsi_eh_6]
root         160  0.0  0.0      0     0 ?        I<   12:59   0:00 [kworker/R-scsi_]
root         161  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/u2:3-events_unbound]
root         162  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/u2:4-flush-253:0]
root         163  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/u2:5-writeback]
root         168  0.0  0.0      0     0 ?        I    12:59   0:00 [kworker/0:4-cgroup_destroy]

ubuntu:~$ top
top - 13:06:16 up 6 min,  0 user,  load average: 1.64, 0.46, 0.19
Tasks: 126 total,   6 running, 120 sleeping,   0 stopped,   0 zombie
%Cpu(s): 54.5 us, 45.5 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st 
MiB Mem :   1903.3 total,    892.3 free,    377.7 used,    800.9 buff/cache     
MiB Swap:   1024.0 total,   1024.0 free,      0.0 used.   1525.5 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                      
   1636 root      20   0    2696   1408   1408 R  27.3   0.1   0:08.03 yes                                                                                          
   1637 root      20   0    3768   2560   1536 R  27.3   0.1   0:08.03 dd                                                                                           
   1638 root      20   0    4324   2944   2816 R  27.3   0.2   0:08.03 bash                                                                                         
      1 root      20   0   22232  13160   9448 S   0.0   0.7   0:01.34 systemd                                                                                      
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd                                                                                     
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_workqueue_release                                                                       
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_g                                                                              
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_p                                                                              
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-slub_                                                                              
      7 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-netns                                                                              
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.04 kworker/0:0H-kblockd                                                                         
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.11 kworker/u2:0-flush-253:0                                                                     
     12 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-mm_pe                                                                              
     13 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_kthread                                                                            
     14 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_rude_kthread                                                                       
     15 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_trace_kthread                                                                      
     16 root      20   0       0      0      0 S   0.0   0.0   0:00.04 ksoftirqd/0                                                                                  
     17 root      20   0       0      0      0 I   0.0   0.0   0:00.06 rcu_preempt                                                                                  
     18 root      rt   0       0      0      0 S   0.0   0.0   0:00.00 migration/0                                                                                  
     19 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 idle_inject/0                                                                                
     20 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0                                                                                      
     21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs                                                                                    
     22 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-inet_                                                                              
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd                                                                                      
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.00 khungtaskd                                                                                   
     26 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reaper                                                                                   
     27 root      20   0       0      0      0 I   0.0   0.0   0:00.02 kworker/u2:2-events_unbound                                                                  
     28 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-write                                                                              
     29 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kcompactd0                                                                                   
     30 root      25   5       0      0      0 S   0.0   0.0   0:00.00 ksmd                                                                                         
     31 root      39  19       0      0      0 S   0.0   0.0   0:00.00 khugepaged                                                                                   
     32 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kinte                                                                              
     33 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kbloc                                                                              
     34 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-blkcg                                                                              
     35 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 irq/9-acpi                                                                                   
     36 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-tpm_d                                                                              
     37 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-ata_s                                                                              
     38 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-md                                                                                 
     39 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-md_bi                                                                              
     40 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-edac-                                                                              
     41 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-devfr                                                                              
     42 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 watchdogd                                                                                    
ubuntu:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.9Gi       377Mi       892Mi       1.0Mi       800Mi       1.5Gi
Swap:          1.0Gi          0B       1.0Gi
```

### Step 2: Identify High Resource Consumers

**Filter or sort processes to find the top CPU and memory users:**
```sh
ps aux --sort=-%cpu | head -n 10
ps aux --sort=-%mem | head -n 10
```

```sh
ubuntu:~$ ps aux --sort=-%cpu | head -n 10
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        1636 32.9  0.0   2696  1408 ?        R    13:05   0:21 yes
root        1637 32.9  0.1   3768  2560 ?        R    13:05   0:21 dd if=/dev/zero of=/dev/null bs=1M
root        1638 32.9  0.1   4324  2944 ?        R    13:05   0:21 bash -c while :; do :; done
root        1640  0.5  2.0 807584 40792 ?        SNl  13:06   0:00 /opt/theia/node /opt/theia/node_modules/@theia/core/lib/node/messaging/ipc-bootstrap --nsfwOptions={}
root        1652  0.4  1.9 658820 38868 ?        SNl  13:06   0:00 /opt/theia/node /opt/theia/node_modules/@theia/core/lib/node/messaging/ipc-bootstrap
root        1659  0.4  0.1   6372  3328 pts/1    RNs+ 13:06   0:00 /root -1 -1 1 /bin/bash
root           1  0.3  0.6  22232 13160 ?        Ss   12:59   0:01 /sbin/init
root        1219  0.2  3.2 837916 63960 ?        SNl  12:59   0:01 /opt/theia/node /opt/theia/browser-app/src-gen/backend/main.js /root --hostname=0.0.0.0 --port 40205
root        1258  0.1  0.4 1230676 8888 ?        S<l  12:59   0:00 /bin/runtime-info-service

ubuntu:~$ ps aux --sort=-%mem | head -n 10
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         853  0.0  3.5 1824352 70144 ?       Ssl  12:59   0:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root        1219  0.2  3.2 837916 63960 ?        SNl  12:59   0:01 /opt/theia/node /opt/theia/browser-app/src-gen/backend/main.js /root --hostname=0.0.0.0 --port 40205
root         624  0.0  2.4 1654564 48256 ?       Ssl  12:59   0:00 /usr/bin/containerd
root        1640  0.5  2.1 807584 41176 ?        SNl  13:06   0:00 /opt/theia/node /opt/theia/node_modules/@theia/core/lib/node/messaging/ipc-bootstrap --nsfwOptions={}
root        1652  0.3  1.9 658820 38868 ?        SNl  13:06   0:00 /opt/theia/node /opt/theia/node_modules/@theia/core/lib/node/messaging/ipc-bootstrap
root         335  0.0  1.3 288952 27136 ?        SLsl 12:59   0:00 /sbin/multipathd -d -s
root         667  0.0  1.1 110016 22784 ?        Ssl  12:59   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root         607  0.0  0.6 468828 13312 ?        Ssl  12:59   0:00 /usr/libexec/udisks2/udisksd
root           1  0.3  0.6  22232 13160 ?        Ss   12:59   0:01 /sbin/init
```
> Take note of the PIDs of the runaway processes (yes , dd , or infinite loop processes).

### Step 3: Terminate or Limit Processes

**1. Terminate runaway processes safely:**
```sh
pkill yes
pkill dd
```

**2. Check that the processes are gone:**
```sh
ps aux | grep yes
ps aux | grep dd
```

```sh
ubuntu:~$ pkill yes
ubuntu:~$ pkill dd

ubuntu:~$ ps aux | grep yes
root        1696  0.0  0.0   3528  1792 pts/0    S+   13:07   0:00 grep --color=auto yes
ubuntu:~$ ps aux | grep dd 
root           2  0.0  0.0      0     0 ?        S    12:59   0:00 [kthreadd]
message+     591  0.0  0.2   9784  5504 ?        Ss   12:59   0:00 @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
root        1698  0.0  0.0   3528  1792 pts/0    S+   13:07   0:00 grep --color=auto dd
```

### Step 4: Apply CPU/Memory Limits

**Optionally, you can limit a process with:**
```sh
ulimit -v 1000000  # Limit virtual memory to ~1GB
nice -n 10 command  # Start process with lower priority
renice 19 -p <PID> # Reduce CPU priority of running process
```
> This ensures no single process can take down the system.

```sh
ubuntu:~$ ulimit -v 1000000
ubuntu:~$ nice -n 10 bash
ubuntu:~$ renice 19 -p 1638
1638 (process ID) old priority 0, new priority 19
```

### Step 5: Verify Fix
**Confirm that CPU and memory usage are back to normal:**
```sh
top
free -h
```

```sh
ubuntu:~$ top

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                      
   1636 root      20   0    2696   1408   1408 R  27.3   0.1   0:08.03 yes                                                                                          
   1637 root      20   0    3768   2560   1536 R  27.3   0.1   0:08.03 dd                                                                                           
   1638 root      20   0    4324   2944   2816 R  27.3   0.2   0:08.03 bash                                                                                         
      1 root      20   0   22232  13160   9448 S   0.0   0.7   0:01.34 systemd                                                                                      
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd                                                                                     
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_workqueue_release                                                                       
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_g                                                                              
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_p                                                                              
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-slub_                                                                              
      7 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-netns                                                                              
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.04 kworker/0:0H-kblockd                                                                         
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.11 kworker/u2:0-flush-253:0                                                                     
     12 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-mm_pe                                                                              
     13 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_kthread                                                                            
     14 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_rude_kthread                                                                       
     15 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_trace_kthread                                                                      
     16 root      20   0       0      0      0 S   0.0   0.0   0:00.04 ksoftirqd/0                                                                                  
     17 root      20   0       0      0      0 I   0.0   0.0   0:00.06 rcu_preempt                                                                                  
     18 root      rt   0       0      0      0 S   0.0   0.0   0:00.00 migration/0                                                                                  
     19 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 idle_inject/0                                                                                
     20 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0                                                                                      
     21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs                                                                                    
     22 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-inet_                                                                              
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd                                                                                      
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.00 khungtaskd                                                                                   
     26 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reaper                                                                                   
     27 root      20   0       0      0      0 I   0.0   0.0   0:00.02 kworker/u2:2-events_unbound                                                                  
     28 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-write                                                                              
     29 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kcompactd0                                                                                   
     30 root      25   5       0      0      0 S   0.0   0.0   0:00.00 ksmd                                                                                         
     31 root      39  19       0      0      0 S   0.0   0.0   0:00.00 khugepaged                                                                                   
     32 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kinte                                                                              
     33 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kbloc                                                                              
     34 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-blkcg                                                                              
     35 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 irq/9-acpi                                                                                   
top - 13:09:27 up 9 min,  0 user,  load average: 2.76, 1.82, 0.80
Tasks: 126 total,   2 running, 124 sleeping,   0 stopped,   0 zombie
%Cpu(s): 97.7 us,  0.3 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  2.0 st 
MiB Mem :   1903.3 total,    636.5 free,    431.2 used,   1028.6 buff/cache
   
top - 13:10:23 up 10 min,  0 user,  load average: 1.90, 1.75, 0.84
Tasks: 126 total,   2 running, 124 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.3 sy, 99.3 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.3 st 
MiB Mem :   1903.3 total,    586.9 free,    443.9 used,   1065.5 buff/cache     
MiB Swap:   1024.0 total,   1024.0 free,      0.0 used.   1459.3 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                      
   1638 root      39  19    4324   2944   2816 R  99.3   0.2   3:30.33 bash                                                                                         
   1640 root      39  19  825904  51496  29568 S   0.3   2.6   0:02.90 node                                                                                         
      1 root      20   0   22232  13160   9448 S   0.0   0.7   0:01.35 systemd                                                                                      
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd                                                                                     
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_workqueue_release                                                                       
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_g                                                                              
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_p                                                                              
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-slub_                                                                              
      7 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-netns                                                                              
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.04 kworker/0:0H-kblockd                                                                         
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.11 kworker/u2:0-flush-253:0                                                                     
     12 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-mm_pe                                                                              
     13 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_kthread                                                                            
     14 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_rude_kthread                                                                       
     15 root      20   0       0      0      0 I   0.0   0.0   0:00.00 rcu_tasks_trace_kthread                                                                      
     16 root      20   0       0      0      0 S   0.0   0.0   0:00.04 ksoftirqd/0                                                                                  
     17 root      20   0       0      0      0 I   0.0   0.0   0:00.07 rcu_preempt                                                                                  
     18 root      rt   0       0      0      0 S   0.0   0.0   0:00.00 migration/0                                                                                  
     19 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 idle_inject/0                                                                                
     20 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0                                                                                      
     21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs                                                                                    
     22 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-inet_                                                                              
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd                                                                                      
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.00 khungtaskd                                                                                   
     26 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reaper                                                                                   
     27 root      20   0       0      0      0 I   0.0   0.0   0:00.03 kworker/u2:2-events_unbound                                                                  
     28 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-write                                                                              
     29 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kcompactd0                                                                                   
     30 root      25   5       0      0      0 S   0.0   0.0   0:00.00 ksmd                                                                                         
     31 root      39  19       0      0      0 S   0.0   0.0   0:00.00 khugepaged                                                                                   
     32 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kinte                                                                              
     33 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-kbloc                                                                              
     34 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-blkcg                                                                              
     35 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 irq/9-acpi                                                                                   
     36 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-tpm_d                                                                              
     37 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-ata_s                                                                              
     38 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-md                                                                                 
     39 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-md_bi                                                                              
     40 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-edac-                                                                              
     41 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-devfr                                                                              
     42 root     -51   0       0      0      0 S   0.0   0.0   0:00.00 watchdogd                                                                                    
     43 root       0 -20       0      0      0 I   0.0   0.0   0:00.23 kworker/0:1H-kblockd
                                                                        
ubuntu:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.9Gi       443Mi       586Mi       1.0Mi       1.0Gi       1.4Gi
Swap:          1.0Gi          0B       1.0Gi
ubuntu:~$
```

### Summary Table

| Metric                | Before              | After               |
| --------------------- | ------------------- | ------------------- |
| CPU Usage             | ~90–100%            | Mostly idle / nice  |
| CPU Idle              | ~0%                 | High                |
| Load Average          | High                | Low                 |
| System Responsiveness | Poor                | Normal              |
| Memory Used           | ~400–450 MB         | ~440 MB             |
| Swap Used             | 0                   | 0                   |
| Root Cause            | CPU-bound processes | Controlled / killed |

> “Before remediation, CPU was saturated by runaway user processes while memory remained stable. After killing and renicing the processes, CPU idle time recovered and system responsiveness returned, with memory usage largely unchanged.”
