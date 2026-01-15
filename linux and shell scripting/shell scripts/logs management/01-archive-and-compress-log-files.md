## Archive and Compress Log Files Older Than 7 Days / Size of 1M (or) 1k

**Purpose:**
  1. Archive and compress log files on a Linux system to save disk space.
  2. Move the compressed logs to a dedicated archive folder for easier management.

**Use:**
  - `find` → To filter files by size or age.
  - `gzip` → To compress files.
  - `mv` → To move compressed files into a dedicated archive folder.
  - `mkdir -p` → To ensure the archive folder exists before moving files.

**Use Cases:**
  - Reduce disk space used by large log files.
  - Keep a backup/archive of logs for auditing or troubleshooting.
  - Organize logs by moving old/compressed logs to a separate folder.
  - Can be scheduled as a cron job to automate log archiving.

**Tips:**
  - Change -size +1k → -size +1M for larger logs.
  - Change -mtime +7 → -mtime +30 to archive logs older than 30 days.
  - Always check the archive folder to ensure important logs are not accidentally deleted.

```sh
ubuntu:~$ vi archive-logs.sh
ubuntu:~$ chmod +x archive-logs.sh 
ubuntu:~$ cat archive-logs.sh 
#!/bin/bash

file=/var/log/auth.log
arc=/var/log/arc-logs
mkdir -p "$arc"
find "$file" -type f -size +1k  -exec gzip {} \; -exec mv {}.gz "$arc" \;

ubuntu:~$ cd /var/log/
ubuntu:/var/log$ ls -ltrah
total 1.3M
drwxr-xr-x   2 root      root            4.0K Sep  5  2024 dist-upgrade
lrwxrwxrwx   1 root      root              39 Jan 22  2025 README -> ../../usr/share/doc/systemd/README.logs
drwxr-xr-x   2 landscape landscape       4.0K Jan 22  2025 landscape
drwxr-sr-x+  3 root      systemd-journal 4.0K Feb 10  2025 journal
drwx------   2 root      root            4.0K Feb 10  2025 private
-rw-r-----   1 root      adm              15K Feb 10  2025 dmesg.2.gz
-rw-rw----   1 root      utmp             384 Feb 10  2025 btmp.1
-rw-rw-r--   1 root      utmp             292 Feb 10  2025 lastlog
-rw-r-----   1 root      adm              15K Dec 18 17:13 dmesg.1.gz
-rw-r-----   1 root      adm              59K Dec 18 17:14 dmesg.0
drwxr-xr-x  12 root      root            4.0K Dec 18 17:15 ..
-rw-r--r--   1 root      root             15K Dec 18 17:16 alternatives.log.1
-rw-r-----   1 syslog    adm             141K Dec 18 17:16 kern.log.1
-rw-r--r--   1 root      root            250K Dec 18 17:16 dpkg.log.1
-rw-r-----   1 syslog    adm             389K Dec 18 17:17 syslog.1
-rw-r-----   1 syslog    adm              15K Dec 18 17:17 auth.log.1
drwxr-xr-x   2 root      root            4.0K Jan 14 04:33 sysstat
-rw-r--r--   1 root      root               0 Jan 14 04:33 alternatives.log
-rw-r--r--   1 root      root               0 Jan 14 04:33 dpkg.log
-rw-rw----   1 root      utmp               0 Jan 14 04:33 btmp
drwxr-xr-x   2 root      root            4.0K Jan 14 04:33 apt
-rw-rw-r--   1 root      utmp            9.4K Jan 14 04:33 wtmp
-rw-r-----   1 root      adm              56K Jan 14 04:33 dmesg
-rw-r-----   1 syslog    adm              65K Jan 14 04:33 kern.log
drwxr-xr-x   2 root      root            4.0K Jan 14 04:49 killercoda
drwxr-x---   2 root      adm             4.0K Jan 14 05:04 unattended-upgrades
drwxr-xr-x   2 root      root            4.0K Jan 14 05:04 archive
-rw-r-----   1 syslog    adm             145K Jan 14 05:22 syslog
drwxr-xr-x   2 root      root            4.0K Jan 14 05:22 arc-logs
drwxrwxr-x  12 root      syslog          4.0K Jan 14 05:22 .

ubuntu:/var/log$ cd arc-logs/
ubuntu:/var/log/arc-logs$ ls
auth.log.gz

ubuntu:~$ vi archive-logs.sh 
ubuntu:~$ cat archive-logs.sh 
#!/bin/bash

file=/var/log/kern.log
arc=/var/log/arc-logs1
mkdir -p "$arc"
find "$file" -type f -mtime 0 -exec gzip {} \; -exec mv {}.gz "$arc" \;

ubuntu:~$ ./archive-logs.sh 

ubuntu:~$ cd /var/log/
ubuntu:/var/log$ ls 
README              apt        archive     btmp.1        dmesg.0     dpkg.log    kern.log.1  lastlog  syslog.1             wtmp
alternatives.log    arc-logs   auth.log.1  dist-upgrade  dmesg.1.gz  dpkg.log.1  killercoda  private  sysstat
alternatives.log.1  arc-logs1  btmp        dmesg         dmesg.2.gz  journal     landscape   syslog   unattended-upgrades

ubuntu:/var/log$ ls -lh
total 1.2M
lrwxrwxrwx  1 root      root              39 Jan 22  2025 README -> ../../usr/share/doc/systemd/README.logs
-rw-r--r--  1 root      root               0 Jan 14 04:33 alternatives.log
-rw-r--r--  1 root      root             15K Dec 18 17:16 alternatives.log.1
drwxr-xr-x  2 root      root            4.0K Jan 14 04:33 apt
drwxr-xr-x  2 root      root            4.0K Jan 14 05:22 arc-logs
drwxr-xr-x  2 root      root            4.0K Jan 14 05:33 arc-logs1
drwxr-xr-x  2 root      root            4.0K Jan 14 05:04 archive
-rw-r-----  1 syslog    adm              15K Dec 18 17:17 auth.log.1
-rw-rw----  1 root      utmp               0 Jan 14 04:33 btmp
-rw-rw----  1 root      utmp             384 Feb 10  2025 btmp.1
drwxr-xr-x  2 root      root            4.0K Sep  5  2024 dist-upgrade
-rw-r-----  1 root      adm              56K Jan 14 04:33 dmesg
-rw-r-----  1 root      adm              59K Dec 18 17:14 dmesg.0
-rw-r-----  1 root      adm              15K Dec 18 17:13 dmesg.1.gz
-rw-r-----  1 root      adm              15K Feb 10  2025 dmesg.2.gz
-rw-r--r--  1 root      root               0 Jan 14 04:33 dpkg.log
-rw-r--r--  1 root      root            250K Dec 18 17:16 dpkg.log.1
drwxr-sr-x+ 3 root      systemd-journal 4.0K Feb 10  2025 journal
-rw-r-----  1 syslog    adm             141K Dec 18 17:16 kern.log.1
drwxr-xr-x  2 root      root            4.0K Jan 14 04:49 killercoda
drwxr-xr-x  2 landscape landscape       4.0K Jan 22  2025 landscape
-rw-rw-r--  1 root      utmp             292 Feb 10  2025 lastlog
drwx------  2 root      root            4.0K Feb 10  2025 private
-rw-r-----  1 syslog    adm             145K Jan 14 05:30 syslog
-rw-r-----  1 syslog    adm             389K Dec 18 17:17 syslog.1
drwxr-xr-x  2 root      root            4.0K Jan 14 04:33 sysstat
drwxr-x---  2 root      adm             4.0K Jan 14 05:04 unattended-upgrades
-rw-rw-r--  1 root      utmp            9.4K Jan 14 04:33 wtmp

ubuntu:/var/log$ cd arc-logs1/
ubuntu:/var/log/arc-logs1$ ls
kern.log.gz
ubuntu:/var/log/arc-logs1$ 
ubuntu:/var/log/arc-logs1$ cat kern.log.gz 
gikern.log\ksȒ<+2f?
                   npc1mD!@BMͬ/%<ɬ*
˪i-Mlixp {/                       ͨW4lU-v[
Y_֌        ~jjVǞ
[:u5(=X)<*sۮU(IzZEYugajI
                        nwD(#00NMϦ^Czp2Mg$p@@7[_UI
WެkW7\.Yvv2:zՒ{aff:VŬ;z̜Jr4f5`Ak}fx׽M#$Yq,iXϋ)2칸y:YAxW97F-,޼^ObJ

                                                               P1s޿W8\yNOX'|udz
(DkS4bxŝs}-y<x4F~,vw<7RC<z\     쟄;;b<3zRй@_r;p=T
                                                 Snh/}/hc40-l5HOq1c4f~6},1TV+)g~fhFQ:!Q~-jYւ
sulSI6?[jTbۺ7ם[iY^kjnZU                                                                     LYn:D!Lj/΄^]7)̺V
4Q򩁪5NM
      WIm-TZl9
              n
               vϭCmRؖ2\탱X^!*>KiF"l`>67SKWpd4L@;=@
;bG;&R8\}_H*>fg΄xz,fVi|mtP1) gMs
                                -2vtoZ^_
<?i

```
