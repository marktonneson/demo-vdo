## VDO Demo WalkThru

### Requirements
* Minimum VM: 1vCPU x 1G mem, running RHEL 8.latest or RHEL 7.latest
* Will need root or sudo to install packages
* Two extra disks (5GB sufficient)

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
[root@server ~]# vdo create --name=myvdo1 --device=/dev/vdb --vdoLogicalSize=2T
```
Use lsblk to see that you have a new block device of 2TB:
```
[root@server ~]# lsblk
```

Create an XFS file system on the VDO volume:
We use -K because we are not passing discards down to VDO
```
[root@server ~]# mkfs.xfs -K /dev/mapper/myvdo1
```
Mount file system:
```
[root@server ~]# mkdir /mnt/vdo1
[root@server ~]# mount -o discard /dev/mapper/myvdo1 /mnt/vdo1
```

Verify that vdo.service is enabled. It should be enabled by default; if it is not enabled, enable it as it is required by VDO:
```
[root@server ~]# systemctl is-enabled vdo
```

Use vdo status:
```
[root@server ~]# vdo status | grep -i deduplication
[root@server ~]# vdo status | grep -i compression
```

Create a 1GB file:
```
[root@server ~]# dd if=/dev/urandom of=/root/bigfile bs=1024 count=1000000
```

Make note of df and vdostats output:
```
[root@server ~]# df -h /mnt/vdo1
[root@server ~]# vdostats --human-readable
```

Use this loop to make five more copies of the same file in /mnt/vdo1/
```
[root@server ~]# for i in `seq 2 6`; do rsync --progress /root/bigfile /mnt/vdo1/file$i; done
```

View differences in df and vdostats output:
```
[root@server ~]# df -h /mnt/vdo1
[root@server ~]# vdostats --human-readable
```

Verify the copies on the VDO volume are identical to the original:
```
[root@server ~]# diff -s /root/bigfile /mnt/vdo1/file6
```

##### Grow a VDO Volume
Unmount and remove the VDO volume created previously:
```
[root@server ~]# umount /mnt/vdo1/
[root@server ~]# vdo remove --all
```
Create a VDO volume on top of an LVM logical volume:
```
[root@server ~]# pvcreate /dev/vdb
[root@server ~]# vgcreate myvg /dev/vdb
[root@server ~]# lvcreate -l 100%FREE -n mylv myvg
[root@server ~]# vdo create --name=growvdo --device=/dev/mapper/myvg-mylv --vdoLogicalSize=1T
```
Create an XFS file system on the VDO volume and mount it:
```
[root@server ~]# mkfs.xfs -K /dev/mapper/growvdo
[root@server ~]# mkdir /mnt/growvdo
[root@server ~]# mount -o discard /dev/mapper/growvdo /mnt/growvdo/
```
Verify the disk size:
```
[root@server ~]# df -h /mnt/growvdo/
[root@server ~]# vdostats --human-readable
```
Increase the logical size of the VDO volume from 1TB to 2TB:
```
[root@server ~]# vdo growLogical --name=growvdo --vdoLogicalSize=2T
```
Expand the XFS file system, and verify that the size has increased to 2TB:
```
[root@server ~]# xfs_growfs /mnt/growvdo/
```
Increase the physical size of the VDO volume by extending the underlying PV, VG, and LV:
```
[root@server ~]# pvcreate /dev/vdc
[root@server ~]# vgextend myvg /dev/vdc
[root@server ~]# lvextend -l +100%FREE /dev/mapper/myvg-mylv
```
Increase the physical size of the VDO volume:
```
[root@server ~]# vdo growPhysical --name=growvdo
```
Verify the size has increased:
```
[root@server ~]# vdostats --human-readable
```

* Cleanup
  * Optional if demo VM is disposable
```
# umount /mnt/growvdo
# vdo remove --all
# rm /root/bigfile
# rmdir /mnt/growvdo /mnt/vdo1
```
