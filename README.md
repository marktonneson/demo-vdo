# demo-VDO

Quick demo of Virtual Data Optimizer (VDO).

Virtual Data Optimizer (VDO) provides inline data reduction for Linux in the form of deduplication, compression, and thin provisioning. When you set up a VDO volume, you specify a block device on which to construct your VDO volume and the amount of logical storage you plan to present.

### Notes
VDO has two kernel modules that work together to provide data efficiency
* **kvdo** - implements block virtualization and compression
* **uds** - maintains the deduplication index and provides hash advice

Can be used in any storage environment, including:
* Block storage - local, LVM, Ceph, iSCSI, SCSI
* File storage - local, Gluster, NFS, CIFS
* Object storage - Ceph, Swift

##### VDO Utilities:
* **vdo** - manage VDO volumes (create, remove, modify) and view configuration (status)
* **vdostats** - provides information on the volumes' statistics and health

##### Benefits of Using VDO
* VDO increases storage efficiency and reduces costs.
* VDO makes existing high-performance storage more affordable with minimal impact on performance.
* Because data reduction is applied by the OS, VDO works anywhere that the OS resides (on-premises or in public, private or hybrid cloud).
* OS-based data reduction benefits software layers above the OS, including applications and infrastructure products such as Ceph, Gluster, virtualization, containers, etc.
* Copying a VDO volume (such as when using dd or a replication tool like DRBD):
 * Consumes less network bandwidth
 * Takes less time

##### Deployment Precautions
 * DO NOT put RAID on top of VDO
 * DO NOT put thin provisioning under VDO

### Installation
Install packages:
```
[root@server ~]# yum install vdo kmod-kvdo
```
Add the kvdo module in the kernel:
```
[root@server ~]# modprobe kvdo
```

### Configuration
On RHEL 8, enable periodic block discards:
```
[root@server ~]# systemctl enable --now fstrim.timer
```
Disable deduplication on a VDO volume with vdo disableDeduplication:
```
[root@server ~]# vdo disableDeduplication --name=myvdo1
```
Enable deduplication on a VDO volume with vdo enableDeduplication:
```
[root@server ~]# vdo enableDeduplication --name=myvdo1
```
Disable compression on a VDO volume with vdo disableCompression:
```
[root@server ~]# vdo disableCompression --name=myvdo1
```
Enable compression on a VDO volume with vdo enableCompression:
```
[root@server ~]# vdo enableCompression --name=myvdo1
```


### Testing
View VDO statistics:
```
[root@server ~]# vdostats
[root@server ~]# vdostats --human-readable
[root@server ~]# vdostats --verbose
```

##### Troubleshooting - Recover a VDO Volume
You can recover a VDO volume using the forceRebuild option for vdo.

You may need to recover a VDO volume if:
* It fails to start
* It enters read-only mode

Possible reasons for these failure:
* Corruption
* I/O errors from storage layers below VDO

Perform a recovery using the forcedRebuild option for vdo:
```
[root@server ~]# vdo stop --name=myvdo1
[root@server ~]# vdo start --name=myvdo1 --forceRebuild
[root@server ~]# vdostats --verbose|grep "operating mode"
  operating mode                      : normal
```
Check the number of times a recovery has been performed:
```
[root@server ~]# vdostats --verbose|grep "read-only recovery"
  read-only recovery count            : 1
```

### References and Resources
* [RHEL 9 Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/deduplicating_and_compressing_logical_volumes_on_rhel/index)
* [RHEL 8 Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deduplicating_and_compressing_storage/index)
* [RHEL 7 Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/vdo)
* [Red Hat Labs - VDO](https://lab.redhat.com/vdo-configure)
