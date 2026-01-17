## Exploring Mounting and Unmounting Disks in Linux

**In this scenario, you’ll learn how to:**
  - List block devices and partitions.
  - Mount a disk to a directory.
  - Check mounted filesystems.
  - Unmount a disk safely.
    
**You’ll use the following commands:**
  - `lsblk` – list block devices
  - `blkid` – display block device attributes
  - `df -h` – show disk usage
  - `mount / umount` – mount and unmount filesystems
  - `mkdir` – create mount points

> No issues or troubleshooting needed here — this scenario focuses purely on understanding how disk mounting works

 ### Step 1: Inspect Available Disks
 
```sh
# ---------------  Listing the block devices attached to the system ---------------

ubuntu:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0    1G  0 loop 
vda     253:0    0   20G  0 disk 
|-vda1  253:1    0   19G  0 part /
|-vda14 253:14   0    4M  0 part 
|-vda15 253:15   0  106M  0 part /boot/efi
`-vda16 259:0    0  913M  0 part /boot

# --------------- Filesystem types and UUIDs ---------------
ubuntu:~$ blkid
/dev/vda15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="F137-8D01" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="9ac3ca4b-722a-4849-a2ec-2094c08b6754"
/dev/vda1: LABEL="cloudimg-rootfs" UUID="504cde81-04d8-46d1-9dbd-78fecd7497f4" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="f0befb13-d602-4785-b4ee-3f044a9f1b7f"
/dev/vda16: LABEL="BOOT" UUID="9f27aac6-c8cb-412a-956b-2663fc38484c" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="0e511cae-a17a-4fc2-b7a7-587963b39d06"
/dev/loop0: UUID="b77628e9-f1cf-407d-922a-778899189e3f" BLOCK_SIZE="4096" TYPE="ext4"
/dev/vda14: PARTUUID="55719d8e-850b-48b2-ac8d-e92326bc7e36"

# --------------- check space usage on mounted disks ---------------
ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  992K  190M   1% /run
/dev/vda1        19G  5.2G   14G  29% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi
```

### Step 2: Mount a Disk

```sh
# --------------- Create a mount point ---------------
ubuntu:~$ sudo mkdir /mnt/data

# --------------- Then, mount the disk ---------------
ubuntu:~$ sudo mount /dev/loop0 /mnt/data

# --------------- Verify the mount ---------------
ubuntu:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0    1G  0 loop /mnt/data
vda     253:0    0   20G  0 disk 
|-vda1  253:1    0   19G  0 part /
|-vda14 253:14   0    4M  0 part 
|-vda15 253:15   0  106M  0 part /boot/efi
`-vda16 259:0    0  913M  0 part /boot

# --------------- Verify the mounted disk and its available space. ---------------
ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  992K  190M   1% /run
/dev/vda1        19G  5.2G   14G  29% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi
/dev/loop0      974M   24K  907M   1% /mnt/data
```

### Step 3: Unmount and Verify

```sh
# --------------- Unmount the disk ---------------

ubuntu:~$ sudo umount /mnt/data

# --------------- Verify the unmount ---------------
ubuntu:~$ lsblk 
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0    1G  0 loop 
vda     253:0    0   20G  0 disk 
|-vda1  253:1    0   19G  0 part /
|-vda14 253:14   0    4M  0 part 
|-vda15 253:15   0  106M  0 part /boot/efi
`-vda16 259:0    0  913M  0 part /boot

ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  988K  190M   1% /run
/dev/vda1        19G  5.2G   14G  29% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi
ubuntu:~$
```
