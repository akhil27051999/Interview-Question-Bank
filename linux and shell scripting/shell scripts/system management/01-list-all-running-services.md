## List out all the running services in the system

**Purpose:**
  1. List all currently running services on a Linux system using systemctl.
  2. Demonstrate service management commands and unit states.
  3. Useful for system monitoring, troubleshooting, and auditing active services.

**Use Cases:**
  - `System administration`: Check which services are running on production or development servers.
  - `Troubleshooting`: Quickly identify if critical services (SSH, Docker, network services) are active.
  - `Monitoring & audits`: Can be scheduled in cron jobs or monitoring scripts to log service status.
  - `DevOps`: Use as part of automated health checks or CI/CD pipelines.
    
```sh

ubuntu:~$ systemctl list-jobs
No jobs running.

ubuntu:~$ systemctl list-units
  UNIT                                                                                     LOAD   ACTIVE SUB       DESCRIPTION                                     >
  proc-sys-fs-binfmt_misc.automount                                                        loaded active running   Arbitrary Executable File Formats File System Au>
  sys-devices-pci0000:00-0000:00:02.0-0000:01:00.0-virtio0-net-enp1s0.device               loaded active plugged   Virtio 1.0 network device
  sys-devices-pci0000:00-0000:00:02.5-0000:06:00.0-virtio2-virtio\x2dports-vport2p1.device loaded active plugged   /sys/devices/pci0000:00/0000:00:02.5/0000:06:00.>
  sys-devices-pci0000:00-0000:00:02.6-0000:07:00.0-virtio3-block-vda-vda1.device           loaded active plugged   /sys/devices/pci0000:00/0000:00:02.6/0000:07:00.>
  sys-devices-pci0000:00-0000:00:02.6-0000:07:00.0-virtio3-block-vda-vda14.device          loaded active plugged   /sys/devices/pci0000:00/0000:00:02.6/0000:07:00.>
  sys-devices-pci0000:00-0000:00:02.6-0000:07:00.0-virtio3-block-vda-vda15.device          loaded active plugged   /sys/devices/pci0000:00/0000:00:02.6/0000:07:00.>
  sys-devices-pci0000:00-0000:00:02.6-0000:07:00.0-virtio3-block-vda-vda16.device          loaded active plugged   /sys/devices/pci0000:00/0000:00:02.6/0000:07:00.>
  sys-devices-pci0000:00-0000:00:02.6-0000:07:00.0-virtio3-block-vda.device                loaded active plugged   /sys/devices/pci0000:00/0000:00:02.6/0000:07:00.>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.1-tty-ttyS1.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.10-tty-ttyS10.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.11-tty-ttyS11.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.12-tty-ttyS12.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.13-tty-ttyS13.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.14-tty-ttyS14.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.15-tty-ttyS15.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.16-tty-ttyS16.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.17-tty-ttyS17.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.18-tty-ttyS18.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.19-tty-ttyS19.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.2-tty-ttyS2.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.20-tty-ttyS20.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.21-tty-ttyS21.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.22-tty-ttyS22.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.23-tty-ttyS23.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.24-tty-ttyS24.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.25-tty-ttyS25.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.26-tty-ttyS26.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.27-tty-ttyS27.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.28-tty-ttyS28.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.29-tty-ttyS29.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.3-tty-ttyS3.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.30-tty-ttyS30.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.31-tty-ttyS31.device           loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.4-tty-ttyS4.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.5-tty-ttyS5.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.6-tty-ttyS6.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.7-tty-ttyS7.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.8-tty-ttyS8.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-platform-serial8250-serial8250:0-serial8250:0.9-tty-ttyS9.device             loaded active plugged   /sys/devices/platform/serial8250/serial8250:0/se>
  sys-devices-pnp0-00:00-00:00:0-00:00:0.0-tty-ttyS0.device                                loaded active plugged   /sys/devices/pnp0/00:00/00:00:0/00:00:0.0/tty/tt>
  sys-devices-virtual-misc-rfkill.device                                                   loaded active plugged   /sys/devices/virtual/misc/rfkill
  sys-devices-virtual-net-docker0.device                                                   loaded active plugged   /sys/devices/virtual/net/docker0
  sys-devices-virtual-tty-ttyprintk.device                                                 loaded active plugged   /sys/devices/virtual/tty/ttyprintk
  sys-module-configfs.device                                                               loaded active plugged   /sys/module/configfs
  sys-module-fuse.device                                                                   loaded active plugged   /sys/module/fuse
  sys-subsystem-net-devices-docker0.device                                                 loaded active plugged   /sys/subsystem/net/devices/docker0
  sys-subsystem-net-devices-enp1s0.device                                                  loaded active plugged   Virtio 1.0 network device                       >

ubuntu:~$ vi list-out-running-services.sh
ubuntu:~$ cat list-out-running-services.sh

#!/bin/bash

echo "List out all the running services on the system"
echo "-----------------------------------------------"

systemctl list-units --type=service --state=running

ubuntu:~$ chmod +x list-out-running-services.sh

ubuntu:~$ ./list-out-running-services.sh 
List out all the running services on the system
-----------------------------------------------
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION                                   
  containerd.service          loaded active running containerd container runtime
  cron.service                loaded active running Regular background program processing daemon
  dbus.service                loaded active running D-Bus System Message Bus
  docker.service              loaded active running Docker Application Container Engine
  fwupd.service               loaded active running Firmware update daemon
  getty@tty1.service          loaded active running Getty on tty1
  ModemManager.service        loaded active running Modem Manager
  multipathd.service          loaded active running Device-Mapper Multipath Device Controller
  polkit.service              loaded active running Authorization Manager
  rsyslog.service             loaded active running System Logging Service
  serial-getty@ttyS0.service  loaded active running Serial Getty on ttyS0
  ssh.service                 loaded active running OpenBSD Secure Shell server
  systemd-journald.service    loaded active running Journal Service
  systemd-logind.service      loaded active running User Login Management
  systemd-networkd.service    loaded active running Network Configuration
  systemd-resolved.service    loaded active running Network Name Resolution
  systemd-timesyncd.service   loaded active running Network Time Synchronization
  systemd-udevd.service       loaded active running Rule-based Manager for Device Events and Files
  udisks2.service             loaded active running Disk Manager
  unattended-upgrades.service loaded active running Unattended Upgrades Shutdown

Legend: LOAD   → Reflects whether the unit definition was properly loaded.
        ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
        SUB    → The low-level unit activation state, values depend on unit type.

20 loaded units listed.
```
