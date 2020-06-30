## VDO Demo WalkThru

### Requirements
* Minimum VM: 1vCPU x 1G mem, running RHEL 7.latest
* Will need root or sudo to install packages
* Run on server with Desktop/GUI

### WalkThru
* Step One
  * Details
```
[root@server ~]# yum install vdo kmod-kvdo
```
Add the kvdo module in the kernel:
```
[root@server ~]# modprobe kvdo
```

Create a VDO device named myvdo1 on physical device /dev/vdc with a logical size of 2TB:
```
[root@instructor ~]# vdo create --name=myvdo1 --device=/dev/vdc --vdoLogicalSize=2T
```
Use lsblk to see that you have a new block device of 2TB:
```
[root@instructor ~]# lsblk /dev/vdc
```

Create an XFS file system on the VDO volume:
We use -K because we are not passing discards down to VDO
```
[root@instructor ~]# mkfs.xfs -K /dev/mapper/myvdo1
```
Mount file system:
```
[root@instructor ~]# mkdir /mnt/vdo1
[root@instructor ~]# mount -o discard /dev/mapper/myvdo1 /mnt/vdo1
```

Verify that vdo.service is enabled. It should be enabled by default; if it is not enabled, enable it as it is required by VDO:
```
[root@instructor ~]# systemctl is-enabled vdo
```

Use vdo status:
```
[root@instructor ~]# vdo status | grep -i deduplication
[root@instructor ~]# vdo status | grep -i compression
```

Use this loop to make five more copies of the same file in /data/
```
[root@instructor ~]# for i in `seq 2 6`; do rsync --progress curr_udpjitter_2016_07.csv /data/file$i; done
```

View differences in df and vdostats output:
```
[root@instructor ~]# df -h /data/
[root@instructor ~]# vdostats --human-readable
```
Verify the copies on the VDO volume are identical to the original:
```
[root@instructor ~]# diff -s /root/curr_udpjitter_2016_07.csv /vdodata/file6
```
Grow a VDO Volume
Unmount and remove the VDO volume created in the previous lab:
```
[root@instructor ~]# umount /mnt/vdovol/
[root@instructor ~]# umount /data
[root@instructor ~]# vdo remove --all
```
Create a VDO volume on top of an LVM logical volume:
```
[root@instructor ~]# pvcreate /dev/vdc
[root@instructor ~]# vgcreate myvg /dev/vdc
[root@instructor ~]# lvcreate -l 100%FREE -n mylv myvg
[root@instructor ~]# vdo create --name=growvdo --device=/dev/mapper/myvg-mylv --vdoLogicalSize=1T
```
Create an XFS file system on the VDO volume and mount it:
```
[root@instructor ~]# mkfs.xfs -K /dev/mapper/growvdo
[root@instructor ~]# mkdir /mnt/growvdo
[root@instructor ~]# mount -o discard /dev/mapper/growvdo /mnt/growvdo/
```
Verify the disk size:
```
[root@instructor ~]# df -h /mnt/growvdo/
[root@instructor ~]# vdostats --human-readable
```
Increase the logical size of the VDO volume from 1TB to 2TB:
```
[root@instructor ~]# vdo growLogical --name=growvdo --vdoLogicalSize=2T
```
Expand the XFS file system, and verify that the size has increased to 2TB:
```
[root@instructor ~]# xfs_growfs /mnt/growvdo/
```
Increase the physical size of the VDO volume by extending the underlying PV, VG, and LV:
```
[root@instructor ~]# pvcreate /dev/vdd
[root@instructor ~]# vgextend myvg /dev/vdd
[root@instructor ~]# lvextend -l +100%FREE /dev/mapper/myvg-mylv
```
Increase the physical size of the VDO volume:
```
[root@instructor ~]# vdo growPhysical --name=growvdo
```
Verify the size has increased to 25GB:
```
[root@instructor ~]# vdostats --human-readable
```
---
>file contents
>basically anything but code

* Cleanup
  * Run [Tech]-cleanup.sh
  * Optional if demo VM is disposable

Option for multi-scenario demos
#### Scenario: description
* Step One
  * Details
