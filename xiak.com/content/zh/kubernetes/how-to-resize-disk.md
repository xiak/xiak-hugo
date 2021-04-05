---
title: "如何重新划分Linux逻辑分区大小"
date: 2019-06-19T18:30:00+08:00
author: Xiak
image : "images/blog/keyboard.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Kubernetes"]
tags: ["Kubernetes", "DevOps", "Linux"]
description: ""
draft: false
type: "post"
---

# 如何重新划分Linux逻辑分区大小

在工作中遇到过很多因为逻辑分区(Logical Volume)写满导致系统宕机的情况, 这时要么转移数据到其他分区，要么划分更多的磁盘空间给当前逻辑分区, 转移数据的成本是很高的, 那怎么快速，安全给逻辑分区扩容呢?

## 准备工作

本文以 `CentOS7` 为例, 将 `root` 用户的用户空间从 `50G` 扩容到 `950G`, 希望大家理解原理，能够举一反三。 其他 `Linux` 如何重新分区, 增容, 缩容欢迎大家留言

- CentOS7
- xfs 文件系统

## 开始工作

### 查看 disk 状况

执行
```
df -h
```

输出
```bash
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             9.8G     0  9.8G   0% /dev
tmpfs                9.9G     0  9.9G   0% /dev/shm
tmpfs                9.9G   17M  9.8G   1% /run
tmpfs                9.9G     0  9.9G   0% /sys/fs/cgroup
/dev/mapper/cl-root   50G  2.3G   48G   5% /
/dev/sda1           1014M  186M  829M  19% /boot
/dev/mapper/cl-home  963G   33M  963G   1% /home
tmpfs                2.0G     0  2.0G   0% /run/user/0
tmpfs                9.9G   12K  9.9G   1% /var/lib/kubelet/pods/e239e578-3e0d-11e9-bc6f-00505699d344/volumes/kubernetes.io~secret/etcd-certs
tmpfs                9.9G   12K  9.9G   1% /var/lib/kubelet/pods/e239c969-3e0d-11e9-bc6f-00505699d344/volumes/kubernetes.io~secret/kube-proxy-token-d5jd4
tmpfs                9.9G   12K  9.9G   1% /var/lib/kubelet/pods/e239e578-3e0d-11e9-bc6f-00505699d344/volumes/kubernetes.io~secret/calico-node-token-hmtlg
overlay               50G  2.3G   48G   5% /var/lib/docker/overlay2/5628047df066f0294141dabc10969cc5d9931bc560de0149b6c9fc3217749ba8/merged
overlay               50G  2.3G   48G   5% /var/lib/docker/overlay2/75f4076736685f55328e109bff890a98f90dee748bff9514be2f3d339fc21589/merged
shm                   64M     0   64M   0% /var/lib/docker/containers/54f04567eb28d31c15a7cba35cf367b49b92e0a041bd09f93439d2e4c4ff1922/mounts/shm
shm                   64M     0   64M   0% /var/lib/docker/containers/9c2820a58ae955e7e37c2ef9121aba75423d1b1eda9b2e690a44bccdbf84da8c/mounts/shm
overlay               50G  2.3G   48G   5% /var/lib/docker/overlay2/25404b02f6acf7a2415055537963112784b89dab646a617ea06521ede54096d0/merged
overlay               50G  2.3G   48G   5% /var/lib/docker/overlay2/8c532f63cb217a809c5975dade783c32d7848079c755ce30d266b5e2dd05aa05/merged
```

### 备份 /home 目录

执行
```
cp -r /home/ homebak/
```


### 卸载 /home 目录

执行
```
umount /home
```

如果 `/home` 目录下有进程正在运，使用 `fuser -m -v -i -k /home` 终止 `/home` 目录下的进程，最后使用 `umount /home` 卸载 `/home`

### 删除设备

`xfs` 文件系统不支持减少逻辑分区，无法使用 `lvreduce`，这里用 `lvremove` 先删除 `/home` 所在的lv

执行
```
lvremove /dev/mapper/cl-home
```

输出
```bash
Do you really want to remove active logical volume cl/home? [y/n]: y
  Logical volume "home" successfully removed
[root@node1 ~]# lvextend -L +900G /dev/mapper/cl-root
  Size of logical volume cl/root changed from 50.00 GiB (12800 extents) to 950.00 GiB (243200 extents).
  Logical volume cl/root successfully resized.
[root@node1 ~]# xfs_growfs /dev/mapper/cl-root
meta-data=/dev/mapper/cl-root    isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 249036800
```

### 查看 vg 

执行
```
vgdisplay
```

输出
```bash
--- Volume group ---
  VG Name               cl
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1023.00 GiB
  PE Size               4.00 MiB
  Total PE              261887
  Alloc PE / Size       14064 / 54.94 GiB
  Free  PE / Size       247823 / 968.06 GiB
  VG UUID               oSQeGz-finT-nTPd-RvOx-xr2J-2FQh-nkTldl

```

根据 `Free  PE / Size`  来增加 `lv`, 注意， `Free  PE / Size` 一定要留一部分给 `/home`, 这里 我准备留 `68G` 给 `/home`， 其余的都扩展 `/root`

### 扩展 /root所在的lv，增加 900G

执行
```
lvextend -L +900G /dev/mapper/cl-root
```

输出
```bash
Size of logical volume cl/root changed from 50.00 GiB (12800 extents) to 950.00 GiB (243200 extents).
Logical volume cl/root successfully resized.
```

### 扩展 /root文件系统

执行
```
xfs_growfs /dev/mapper/cl-root
```

输出
```bash
meta-data=/dev/mapper/cl-root    isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 249036800
```

### 重建 home 的lv

执行
```
lvcreate -L 63G -n home cl
```

输出

```bash
 Logical volume "home" created.
```

### 查看 vg 

执行
```
vgdisplay
```

输出
```bash
--- Volume group ---
  VG Name               cl
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1023.00 GiB
  PE Size               4.00 MiB
  Total PE              261887
  Alloc PE / Size       261840 / 1022.81 GiB
  Free  PE / Size       47 / 188.00 MiB
  VG UUID               lIOQS4-KiX0-yfvd-5iJO-BqPm-WIIx-7uie2W
```
查看 `Free  PE / Size` 是否被分配完了

### 格式化创建文件系统

执行
```
mkfs.xfs /dev/cl/home
```

输出

```bash
meta-data=/dev/cl/home           isize=512    agcount=4, agsize=4128768 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=16515072, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=8064, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

### 重新挂载 home 目录

执行
```
mount /dev/cl/home /home
```

### 还原 home 目录数据

执行
```
cp -R ./homebak/* /home
```

###　查看磁盘情况

执行
```
df -h
```

输出
```bash
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             9.8G     0  9.8G   0% /dev
tmpfs                9.9G     0  9.9G   0% /dev/shm
tmpfs                9.9G   17M  9.8G   1% /run
tmpfs                9.9G     0  9.9G   0% /sys/fs/cgroup
/dev/mapper/cl-root  950G  2.3G  948G   1% /
/dev/sda1           1014M  186M  829M  19% /boot
tmpfs                2.0G     0  2.0G   0% /run/user/0
tmpfs                9.9G   12K  9.9G   1% /var/lib/kubelet/pods/e239e578-3e0d-11e9-bc6f-00505699d344/volumes/kubernetes.io~secret/etcd-certs
tmpfs                9.9G   12K  9.9G   1% /var/lib/kubelet/pods/e239c969-3e0d-11e9-bc6f-00505699d344/volumes/kubernetes.io~secret/kube-proxy-token-d5jd4
tmpfs                9.9G   12K  9.9G   1% /var/lib/kubelet/pods/e239e578-3e0d-11e9-bc6f-00505699d344/volumes/kubernetes.io~secret/calico-node-token-hmtlg
overlay              950G  2.3G  948G   1% /var/lib/docker/overlay2/5628047df066f0294141dabc10969cc5d9931bc560de0149b6c9fc3217749ba8/merged
overlay              950G  2.3G  948G   1% /var/lib/docker/overlay2/75f4076736685f55328e109bff890a98f90dee748bff9514be2f3d339fc21589/merged
shm                   64M     0   64M   0% /var/lib/docker/containers/54f04567eb28d31c15a7cba35cf367b49b92e0a041bd09f93439d2e4c4ff1922/mounts/shm
shm                   64M     0   64M   0% /var/lib/docker/containers/9c2820a58ae955e7e37c2ef9121aba75423d1b1eda9b2e690a44bccdbf84da8c/mounts/shm
overlay              950G  2.3G  948G   1% /var/lib/docker/overlay2/25404b02f6acf7a2415055537963112784b89dab646a617ea06521ede54096d0/merged
overlay              950G  2.3G  948G   1% /var/lib/docker/overlay2/8c532f63cb217a809c5975dade783c32d7848079c755ce30d266b5e2dd05aa05/merged
/dev/mapper/cl-home   63G   33M   63G   1% /home
```

可以看到 `/dev/mapper/cl-home` 从 `963G` 减少到了 `63G`

`/dev/mapper/cl-root` 从 `50G` 增加到了 `950G`


##　如果新增加硬盘

### 手动给主机增加 100G 硬盘

### 扫描 SCSI总线并添加 SCSI 设备

执行
```
for host in $(ls /sys/class/scsi_host) ; do echo "- - -" > /sys/class/scsi_host/$host/scan; done
```


### 重新扫描 SCSI 总线

执行
```
for scsi_device in $(ls /sys/class/scsi_device/); do echo 1 > /sys/class/scsi_device/$scsi_device/device/rescan; done
```

### 查看已添加的磁盘，能够看到sdb说明添加成功

执行
```
lsblk
```

输出
```bash
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0           2:0    1    4K  0 disk
sda           8:0    0    1T  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0 1023G  0 part
  ├─cl-root 253:0    0  950G  0 lvm  /
  ├─cl-swap 253:1    0  7.9G  0 lvm
  └─cl-home 253:2    0   63G  0 lvm  /home
sdb           8:16   0  100G  0 disk
sr0          11:0    1 1024M  0 rom
```

看到 `sdb 100G` 就是我们新增的硬盘


### 初始化磁盘

执行
```
pvcreate /dev/sdb
```

### 查看物理信息

执行
```
pvdisplay
```

输出
```bash 
 --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               cl
  PV Size               499.00 GiB / not usable 3.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              127743
  Free PE               31
  Allocated PE          127712
  PV UUID               s8zPrU-Ffcy-UJzD-YpQb-lHNe-Gc7K-MynwBK

  "/dev/sdb" is a new physical volume of "100.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name
  PV Size               100.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               aCtCCE-c0Al-2kR1-1AN5-c2Ll-3q07-cFSbFz
```

### 创建逻辑卷组

创建逻辑卷组 `cl2`, 并将 `/dev/sdb` 物理卷加入到这个卷组里

执行
```
vgcreate cl2 /dev/sdb
```

### 查看逻辑卷组

执行
```
vgdisplay
```

输出
```bash
--- Volume group ---
  VG Name               cl
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  6
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               499.00 GiB
  PE Size               4.00 MiB
  Total PE              127743
  Alloc PE / Size       127712 / 498.88 GiB
  Free  PE / Size       31 / 124.00 MiB
  VG UUID               AdQDAa-TS1n-p8yO-1207-klLV-9nLf-CY622o

  --- Volume group ---
  VG Name               cl2
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               100.00 GiB
  PE Size               4.00 MiB
  Total PE              25599
  Alloc PE / Size       0 / 0
  Free  PE / Size       25599 / 100.00 GiB
  VG UUID               MnZkLp-FwDf-jffM-tUu8-Q3KC-XFX7-zIpzCf
```

### 创建逻辑卷

执行
```
lvcreate -L 99G -n home cl2
```

### 格式化

执行
```
mkfs.xfs /dev/cl2/home
```

### 重新挂载 home 目录

执行
```
mount /dev/cl2/home /home
```

## 附录

### 如何开机自动挂载

如何开机自动挂载 `/dev/cl3/ceph` 到 `/ceph` ?

执行
```bash
cat >> /etc/fstab <<EOF
/dev/mapper/cl3-ceph    /ceph                   xfs     defaults        0 0
EOF
```

### 如何清空磁盘数据

执行
`dd if=/dev/zero of=/dev/sdc bs=512 count=1`
