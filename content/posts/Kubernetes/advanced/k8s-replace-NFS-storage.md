---
author: Ryan
title: 生产案例-Kubernetes 替换后端存储
date: 2023-08-14
lastmod: 2023-08-14
tags:
  - Kubernetes
  - NFS
categories:
  - Kubernetes
expirationReminder:
  enable: false
---



使用 Kubernetes NFS Subdir External Provisioner 插件来动态为 Kubernetes 提供 PV（Persistent Volume）卷，并且该插件本身并不提供 NFS 存储，而是依赖于现有的 NFS 服务器来提供存储。
持久卷目录的命名规则为: 







y用 Kubernetes NFS Subdir External Provisioner 插件来动态为 Kubernetes 提供 PV（Persistent Volume）卷，并且该插件本身并不提供 NFS 存储，而是依赖于现有的 NFS 服务器来提供存储。



用 Kubernetes NFS Subdir External Provisioner 插件来动态为 Kubernetes 提供 PV（Persistent Volume）卷，并且该插件本身并不提供 NFS 存储，而是依赖于现有的 NFS 服务器来提供存储。







## 情况说明

业务使用 **Kubernetes NFS Subdir External Provisioner** 插件来动态为 Kubernetes 提供 PV（Persistent Volume）卷，并且该插件本身并不提供 NFS 存储，而是依赖于现有的 NFS 服务器来提供存储。



命名规则 `${namespace}-${pvcName}-${pvName}` 



出现磁盘空间严重不足情况，需要更换存储。

![image-20230901145753230](http://cdn1.ryanxin.live/image-20230901145753230.png)







 项目地址:

https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/blob/master/charts/nfs-subdir-external-provisioner/README.md





Kubernetes 提供了一种动态创建 Persistent Volumes (PV) 的机制，称为 Dynamic Provisioning。这个机制允许集群管理员事先配置 StorageClass，然后用户只需创建 Persistent Volume Claims (PVC)，PV 就会自动根据 [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) 规则动态创建。 



### 示例

```yaml
nfs:
  server: 013f2dsb-4m2d.cn-beijing.extreme.nas.aliyuncs.com    #NFS 服务器的主机名（必填）
  path: /share  #要使用的挂载点的基本路径
  mountOptions: #安装选项（例如“nfs.vers=3”）
    - vers=3
    - noacl
    - nolock
    - proto=tcp
    - rsize=1048576
    - wsize=1048576
    - hard
    - timeo=600
    - retrans=2
    - noresvport

storageClass: 
  name: nfs-client-speed  # 存储类的名称
  defaultClass: false #设置为默认 StorageClass
  allowVolumeExpansion: true #允许扩大卷
  reclaimPolicy: Delete #回收策略
  provisionerName: nfs-client-speed #动态卷分配者名称，必须和创建的"provisioner"变量中设置的name一致
  archiveOnDelete: true ##设置为"false"时删除PVC不会保留数据,"true"则保留数据 
```



### **StorageClass回收策略**

 由 StorageClass 动态创建的 PersistentVolume 会在类的 [reclaimPolicy](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#reclaiming) 字段中指定[回收策略](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#reclaiming)，可以是 Delete 或者 Retain。

 如果 StorageClass 对象被创建时没有指定 reclaimPolicy，它将默认为 Delete。  



#### 回收（Reclaiming）

当用户不再使用其存储卷时，他们可以从 API 中将 PVC 对象删除， 从而允许该资源被回收再利用。PersistentVolume 对象的回收策略告诉集群， 当其被从申领中释放时如何处理该数据卷。 目前，数据卷可以被 Retained（保留）、Recycled（回收）或 Deleted（删除）。

#### 保留（Retain）

回收策略 Retain 使得用户可以手动回收资源。当 PersistentVolumeClaim 对象被删除时，PersistentVolume 卷仍然存在，对应的数据卷被视为"已释放（released）"。 由于卷上仍然存在这前一申领人的数据，该卷还不能用于其他申领。 管理员可以通过下面的步骤来手动回收该卷：

1. 删除 PersistentVolume 对象。与之相关的、位于外部基础设施中的存储资产 （例如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）在 PV 删除之后仍然存在。
2. 根据情况，手动清除所关联的存储资产上的数据。
3. 手动删除所关联的存储资产。

如果你希望重用该存储资产，可以基于存储资产的定义创建新的 PersistentVolume 卷对象。

#### 删除（Delete）

对于支持 Delete 回收策略的卷插件，删除动作会将 PersistentVolume 对象从 Kubernetes 中移除，同时也会从外部基础设施（如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）中移除所关联的存储资产。 动态制备的卷会继承[其 StorageC](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#reclaim-policy)[lass 中设置的回收策略](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#reclaim-policy)， 该策略默认为 Delete。管理员需要根据用户的期望来配置 StorageClass； 否则 PV 卷被创建之后必须要被编辑或者修补。 参阅[更改 PV 卷的回收策略](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/change-pv-reclaim-policy/)。  





## 1.将多块磁盘组合成 LVM卷



### 1.1 安装gdisk工具

```bash
yum install gdisk
```





### 1.2 磁盘分区

使用 gdisk 或其他分区工具（例如 fdisk）来为每个磁盘(sdc,sdd,sde,sdf 共8T)创建分区。

```bash
[root@xx-worker-05 ~]# lsblk
NAME                               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
fd0                                  2:0    1     4K  0 disk
sda                                  8:0    0     1T  0 disk
├─sda1                               8:1    0     1G  0 part /boot
├─sda2                               8:2    0   199G  0 part
│ ├─centos-root                    253:0    0    50G  0 lvm  /
│ ├─centos-swap                    253:1    0   3.9G  0 lvm
│ └─centos-home                    253:2    0 145.1G  0 lvm  /home
└─sda3                               8:3    0   824G  0 part /nfs-server
sdb                                  8:16   0     1T  0 disk
sdc                                  8:32   0     2T  0 disk
sdd                                  8:48   0     2T  0 disk
sde                                  8:64   0     2T  0 disk
sdf                                  8:80   0     2T  0 disk
sr0                                 11:0    1  1024M  0 rom
```



在 `gdisk` 中，你可以使用 `n` 命令创建新的分区，然后按照提示创建分区。重复这个步骤来创建所需数量的分区。

然后，对每个硬盘进行相同的操作。

```bash
Command (? for help): ?
b       back up GPT data to a file
c       change a partition's name
d       delete a partition
i       show detailed information on a partition
l       list known partition types
n       add a new partition
o       create a new empty GUID partition table (GPT)
p       print the partition table
q       quit without saving changes
r       recovery and transformation options (experts only)
s       sort partitions
t       change a partition's type code
v       verify disk
w       write table to disk and exit
x       extra functionality (experts only)
?       print this menu
```



将硬盘模式改为lvm，代码是`8e00 Linux LVM`

```bash
Command (? for help): l
0700 Microsoft basic data  0c01 Microsoft reserved    2700 Windows RE
3000 ONIE boot             3001 ONIE config           4100 PowerPC PReP boot
4200 Windows LDM data      4201 Windows LDM metadata  7501 IBM GPFS
7f00 ChromeOS kernel       7f01 ChromeOS root         7f02 ChromeOS reserved
8200 Linux swap            8300 Linux filesystem      8301 Linux reserved
8302 Linux /home           8400 Intel Rapid Start     8e00 Linux LVM
a500 FreeBSD disklabel     a501 FreeBSD boot          a502 FreeBSD swap
a503 FreeBSD UFS           a504 FreeBSD ZFS           a505 FreeBSD Vinum/RAID
a580 Midnight BSD data     a581 Midnight BSD boot     a582 Midnight BSD swap
a583 Midnight BSD UFS      a584 Midnight BSD ZFS      a585 Midnight BSD Vinum
a800 Apple UFS             a901 NetBSD swap           a902 NetBSD FFS
a903 NetBSD LFS            a904 NetBSD concatenated   a905 NetBSD encrypted
a906 NetBSD RAID           ab00 Apple boot            af00 Apple HFS/HFS+
af01 Apple RAID            af02 Apple RAID offline    af03 Apple label
af04 AppleTV recovery      af05 Apple Core Storage    be00 Solaris boot
bf00 Solaris root          bf01 Solaris /usr & Mac Z  bf02 Solaris swap
bf03 Solaris backup        bf04 Solaris /var          bf05 Solaris /home
bf06 Solaris alternate se  bf07 Solaris Reserved 1    bf08 Solaris Reserved 2
bf09 Solaris Reserved 3    bf0a Solaris Reserved 4    bf0b Solaris Reserved 5
c001 HP-UX data            c002 HP-UX service         ea00 Freedesktop $BOOT
eb00 Haiku BFS             ed00 Sony system partitio  ed01 Lenovo system partit
```





```bash
[root@xx-worker-05 ~]# gdisk /dev/sdc
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/sdc: 4294967296 sectors, 2.0 TiB
Logical sector size: 512 bytes
Disk identifier (GUID): B62A55C1-A5DE-445A-B8CA-AC489E6B49DE
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 4294967262
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048      4294967262   2.0 TiB     8E00  Linux LVM
```



### **1.3 创建物理卷（Physical Volumes）**

```bash
pvcreate /dev/sdc1 /dev/sdd1 /dev/sde1 /dev/sdf1
```

```bash
[root@xx-worker-05 ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <199.00 GiB / not usable 3.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              50943
  Free PE               1
  Allocated PE          50942
  PV UUID               9ALLQS-tNe2-f2zD-WG09-2rn1-xH21-lkXfhm

  --- Physical volume ---
  PV Name               /dev/sdd1
  VG Name               k8s_nfs_data
  PV Size               <2.00 TiB / not usable 2.98 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              524287
  Free PE               0
  Allocated PE          524287
  PV UUID               GRJKYB-DPcn-XE28-YAze-eoDU-526F-XydILb

  --- Physical volume ---
  PV Name               /dev/sdc1
  VG Name               k8s_nfs_data
  PV Size               <2.00 TiB / not usable 2.98 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              524287
  Free PE               0
  Allocated PE          524287
  PV UUID               ulkd9R-aoaU-104h-e2PM-eKl7-bZY3-FCEm3W

  --- Physical volume ---
  PV Name               /dev/sde1
  VG Name               k8s_nfs_data
  PV Size               <2.00 TiB / not usable 2.98 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              524287
  Free PE               0
  Allocated PE          524287
  PV UUID               0hLr2k-IVYf-XRHi-2VSZ-PH6N-Z0ma-OG4A3w

  --- Physical volume ---
  PV Name               /dev/sdf1
  VG Name               k8s_nfs_data
  PV Size               <2.00 TiB / not usable 2.98 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              524287
  Free PE               26210
  Allocated PE          498077
  PV UUID               H1aYjs-pPDV-STyr-IvPe-6RcF-xUC4-eOoWiI
```





### 1.4 **创建卷组（Volume Group）**

```bash
#创建一个卷组，将物理卷添加到卷组中：
vgcreate k8s_nfs_data /dev/sdc1 /dev/sdd1 /dev/sde1 /dev/sdf1
```

```bash
#这里 k8s_nfs_data 是卷组的名称，你可以自己命名。
[root@xx-worker-05 ~]# vgdisplay k8s_nfs_data
  --- Volume group ---
  VG Name               k8s_nfs_data
  System ID
  Format                lvm2
  Metadata Areas        4
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                4
  Act PV                4
  VG Size               <8.00 TiB
  PE Size               4.00 MiB
  Total PE              2097148
  Alloc PE / Size       2070938 / 7.90 TiB
  Free  PE / Size       26210 / 102.38 GiB
  VG UUID               etAQEx-IDaa-soo5-6d5k-9YHo-JpHX-OpG9Oy
```



### **创建逻辑卷（Logical Volume）**：

使用 `lvcreate` 命令创建逻辑卷，并指定大小：

```bash
lvcreate -n k8s_nfs_lvmdata_1 -L 7.9T  k8s_nfs_data
```

这里 `my_lv` 是逻辑卷的名称，`7.9T` 是逻辑卷的大小。

```bash
 --- Logical volume ---
  LV Path                /dev/k8s_nfs_data/k8s_nfs_lvmdata_1
  LV Name                k8s_nfs_lvmdata_1
  VG Name                k8s_nfs_data
  LV UUID                T9NOtS-ElEw-y287-mG41-dzul-b30H-en1fV1
  LV Write Access        read/write
  LV Creation host, time xx-worker-05, 2023-08-18 15:55:29 +0800
  LV Status              available
  # open                 1
  LV Size                7.90 TiB
  Current LE             2070938
  Segments               4
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
```





```bash
[root@xx-worker-05 ~]# lsblk
NAME                               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
fd0                                  2:0    1     4K  0 disk
sda                                  8:0    0     1T  0 disk
├─sda1                               8:1    0     1G  0 part /boot
├─sda2                               8:2    0   199G  0 part
│ ├─centos-root                    253:0    0    50G  0 lvm  /
│ ├─centos-swap                    253:1    0   3.9G  0 lvm
│ └─centos-home                    253:2    0 145.1G  0 lvm  /home
└─sda3                               8:3    0   824G  0 part /nfs-server
sdb                                  8:16   0     1T  0 disk
sdc                                  8:32   0     2T  0 disk
└─sdc1                               8:33   0     2T  0 part
  └─k8s_nfs_data-k8s_nfs_lvmdata_1 253:3    0   7.9T  0 lvm  /nfs-server2
sdd                                  8:48   0     2T  0 disk
└─sdd1                               8:49   0     2T  0 part
  └─k8s_nfs_data-k8s_nfs_lvmdata_1 253:3    0   7.9T  0 lvm  /nfs-server2
sde                                  8:64   0     2T  0 disk
└─sde1                               8:65   0     2T  0 part
  └─k8s_nfs_data-k8s_nfs_lvmdata_1 253:3    0   7.9T  0 lvm  /nfs-server2
sdf                                  8:80   0     2T  0 disk
└─sdf1                               8:81   0     2T  0 part
  └─k8s_nfs_data-k8s_nfs_lvmdata_1 253:3    0   7.9T  0 lvm  /nfs-server2
sr0                                 11:0    1  1024M  0 rom
```







### **1.5 格式化逻辑卷**

格式化逻辑卷以准备使用：

```bash
mkfs.ext4 /dev/k8s_nfs_data/k8s_nfs_lvmdata_1
```

使用的是 ext4 文件系统，你可以根据需要选择其他文件系统。





### **1.6 挂载逻辑卷**

创建一个目录来挂载逻辑卷，并将逻辑卷挂载到该目录：

```bash
mkdir /nfs-server2
mount /dev/k8s_nfs_data/k8s_nfs_lvmdata_1 /nfs-server2
```











## 2. 更换后端存储

```bash
vim /etc/exports
#/nfs-server  *(rw,sync,insecure,no_root_squash)
/nfs-server2 *(rw,sync,insecure,no_root_squash)

systemctl restart nfs
systemctl restart rpcbind
```





### 2.1更改NFS服务器地址



```yaml
vim nfs-client.values
nfs:
  server: 192.168.10.135
  path: /nfs-server
  mountOptions:
    - vers=4
    - minorversion=0
    - rsize=1048576
    - wsize=1048576
    - hard
    - timeo=600
    - retrans=2
    - noresvport

storageClass:
  name: nfs-client
  defaultClass: true
  allowVolumeExpansion: true
  reclaimPolicy: Delete
  provisionerName: nfs-client
  archiveOnDelete: true
```



```bash
helm install -n kube-system -f nfs-client.values.yaml nfs-client rmxc/nfs-subdir-external-provisioner
#-n 命名空间
#-f 指定values文件
#nfs-client helm项目名字
# rmxc/nfs-subdir-external-provisioner CHART文件
```



### 2.2 查看当前pvc

需要删除当前pvc，然后重新自动获取绑定pv

```bash
kubectl delete pvc -n xx-prod pvc-0770e2e6-70c4-4426-9a43-75e2cab94290

#如果卡住直接删除k8s etcd数据库中的记录
$ kubectl patch pv xxx -p '{"metadata":{"finalizers":null}}'

$ kubectl patch pvc xxx -p '{"metadata":{"finalizers":null}}'
```



查看原PVC的数据目录

```bash
[xx@xx-worker-05 nfs-server]$ ll
total 24
drwxrwxrwx  3 root root   37 Nov 19  2021 xx-prod-app-data-pvc-0770e2e6-70c4-4426-9a43-75e2cab94290
drwxrwxrwx 60 root root 8192 May 19 21:26 xx-prod-fe-nginx-data-pvc-b2d082a0-9df3-42a2-8817-dc4eb4a587ca
drwxrwxrwx  2 root root  227 Feb 17  2022 xx-prod-fe-nginx-log-pvc-66e5f78a-e952-4a18-8a9f-6173d7d2485a
drwxrwxrwx 21 root root 4096 Mar 27 15:32 xx-prod-log-pvc-d717b996-7b65-4c23-be9c-fcdc988266be
drwxrwxrwx 15 root root 4096 Aug 22 18:06 xx-prod-mysql57-data-pvc-d4968732-0cf1-4395-9854-fd4581d2906d
drwxrwxrwx  2 root root   45 Nov 14  2021 xx-prod-mysql57-log-pvc-7f831088-8828-4ab5-8e20-03ab5b048cdd
drwxrwxrwx  2 root root   44 Aug 23 09:27 xx-prod-redis-data-redis-master-0-pvc-7f2b4078-a441-400a-b8e1-8046c3999a8f
```



查看新PVC的数据目录

```bash
[xxu@xx-worker-05 nfs-server2]$ ll
drwxrwxrwx  3 root root  4096 Nov 19  2021 xx-prod-app-data-pvc-0770e2e6-70c4-4426-9a43-75e2cab94291
drwxrwxrwx  3 root root  4096 Aug 25 17:04 xx-prod-app-data-pvc-51174db7-fd78-4448-b9fb-21b6f8577e58
drwxrwxrwx  2 root root  4096 Sep  1 14:21 xx-prod-data-redis-0-pvc-45801e2d-282d-47cc-8020-a157fd9b14de
drwxrwxrwx 60 root root  4096 May 19 21:26 xx-prod-fe-nginx-data-pvc-4ef671bb-4cb3-4012-b637-0564b27637f9
drwxrwxrwx  2 root root  4096 Aug 25 09:46 xx-prod-fe-nginx-log-pvc-14f3a9a9-415d-43ea-840b-c39fa5a28ab7
drwxrwxrwx 10 root root  4096 Aug 25 10:32 xx-prod-log-pvc-fe904a2f-a94d-4f5f-8587-a425417d8612
drwxrwxrwx 15 root root  4096 Aug 26 05:23 xx-prod-mysql57-data2-pvc-085dee0e-1c0e-4a0e-9278-d5ea45ab37c6
drwxrwxrwx  2 root root  4096 Nov 14  2021 xx-prod-mysql57-log2-pvc-67bf033b-f1a4-4df0-8f77-59e47539f3d5

```









### 2.3 使用rsync同步数据



停止各服务，将老PVC中的数据同步到新PVC中，Rsync使用守护进程方式后台运行避免数据同步时间过长导致中断。



服务端配置文件

```bash
# /etc/rsyncd: configuration file for rsync daemon mode

# See rsyncd.conf man page for more options.

# configuration example:
 uid = root
 gid = root
 use chroot = no
 max connections = 200
 timeout = 6000
 ignore errors
 read only = false
 list = false
 auth users = rsync_backup
 secrets file = /etc/rsync.passwd
 log file = /var/log/rsyncd.log
# pid file = /var/run/rsyncd.pid
# exclude = lost+found/
# transfer logging = yes
# timeout = 900
# ignore nonreadable = yes
# dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2
 [nfs-backup]
 comment = welcome to backup!
 path = /nfs-server

 [mysql-data-backup]
 comment = welcome to backup!
 path = /nfs-server2/xx-prod-mysql57-data-pvc-d4968732-0cf1-4395-9854-fd4581d2906d

 [mysql-log-backup]
 comment = welcome to backup!
 path = /nfs-server2/xx-prod-mysql57-log-pvc-7f831088-8828-4ab5-8e20-03ab5b048cdd
 [tree-file-backup]
 comment = welcome to backup!
 path = /nfs-server2/xx-prod-app-data-pvc-0770e2e6-70c4-4426-9a43-75e2cab94291
# [ftp]
#        path = /home/ftp
#        comment = ftp export area

```





### 2.4 同步命令

```bash
nohup rsync -avzrogpP --append-verify rsync_backup@127.0.0.1::tree-file-backup /nfs-server2/xx-prod-app-data-pvc-51174db7-fd78-4448-b9fb-21b6f8577e58 > /var/log/rsync-log-treefile.log 2>&1 &
```





### 2.5 查看日志

```bash
 tail -f /var/log/rsync-log-treefile.log
 
tree-file-server-for-pt/storage/high/2022/02/14/86b7f9c882e744a5b51be2c551f24a4b.jpg
         32,827 100%  244.71kB/s    0:00:00 (xfr#138425, ir-chk=149321/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b834f605614e07bccc92feaa9ab687.jpg
         35,251 100%  256.90kB/s    0:00:00 (xfr#138426, ir-chk=149320/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b8ac41b8394c529c00994427078f0f.png
          1,345 100%    9.80kB/s    0:00:00 (xfr#138427, ir-chk=149319/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b8b2ed44dc45c3ae182eccff7b4499.png
          1,722 100%   12.55kB/s    0:00:00 (xfr#138428, ir-chk=149318/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b932ab32e4472b8bb582c160885877.jpg
         34,690 100%  250.94kB/s    0:00:00 (xfr#138429, ir-chk=149317/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b9ad6ecdec45c9846aee66df52769f.png
          1,978 100%   14.31kB/s    0:00:00 (xfr#138430, ir-chk=149316/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86ba0c2fd3a7462597c2c1e688a081d9.jpg
         37,435 100%  270.80kB/s    0:00:00 (xfr#138431, ir-chk=149315/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86ba59e758674905ba39073f647db788.png
          1,184 100%    8.50kB/s    0:00:00 (xfr#138432, ir-chk=149314/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86ba8bf9b31e464fa53b5cef03b29d52.jpg
         31,866 100%  227.15kB/s    0:00:00 (xfr#138433, ir-chk=149313/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86bb4df3f5e8490f8f1eb421706828aa.jpg
         33,953 100%  240.27kB/s    0:00:00 (xfr#138434, ir-chk=149312/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86bbc7d02a5f4076a6a2545f6cfc729d.jpg
         30,784 100%  216.28kB/s    0:00:00 (xfr#138435, ir-chk=149311/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86bc058ff7e847dd9c6b6c7a69d551a8.jpg
```





## 3. 验证pv状态

```bash
[root@xx-master-01 ~]# kubectl get pv
NAME                                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                        STORAGECLASS   REASON   AGE
pv-nfs-client-nfs-subdir-external-provisioner   10Mi       RWO            Retain           Bound    kube-system/pvc-nfs-client-nfs-subdir-external-provisioner                           9d
pvc-085dee0e-1c0e-4a0e-9278-d5ea45ab37c6        50Gi       RWO            Delete           Bound    xx-prod/mysql57-data2                                        nfs-client              9d
pvc-14f3a9a9-415d-43ea-840b-c39fa5a28ab7        8Gi        RWX            Delete           Bound    xx-prod/fe-nginx-log                                         nfs-client              7d6h
pvc-45801e2d-282d-47cc-8020-a157fd9b14de        8Gi        RWO            Delete           Bound    xx-prod/data-redis-0                                         nfs-client              8d
pvc-4ef671bb-4cb3-4012-b637-0564b27637f9        8Gi        RWX            Delete           Bound    xx-prod/fe-nginx-data                                        nfs-client              7d6h
pvc-51174db7-fd78-4448-b9fb-21b6f8577e58        8Gi        RWX            Delete           Bound    xx-prod/app-data                                             nfs-client              7d23h
pvc-67bf033b-f1a4-4df0-8f77-59e47539f3d5        50Gi       RWO            Delete           Bound    xx-prod/mysql57-log2                                         nfs-client              9d
pvc-d4968732-0cf1-4395-9854-fd4581d2906d        50Gi       RWO            Delete           Bound    xx-prod/mysql57-data                                         nfs-client              655d
pvc-fe904a2f-a94d-4f5f-8587-a425417d8612        8Gi        RWX            Delete           Bound    xx-prod/log                                                  nfs-client              8d
```





