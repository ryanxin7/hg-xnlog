

# ceph-common 依赖问题



缺少依赖：

```bash
 ceph-common depends on librbd1 (= 16.2.10-1bionic); however:
  Package librbd1 is not installed.
 ceph-common depends on python3-cephfs (= 16.2.10-1bionic); however:
  Package python3-cephfs is not installed.
 ceph-common depends on python3-ceph-argparse (= 16.2.10-1bionic); however:
  Package python3-ceph-argparse is not installed.
 ceph-common depends on python3-ceph-common (= 16.2.10-1bionic); however:
  Package python3-ceph-common is not installed.
 ceph-common depends on python3-prettytable; however:
  Package python3-prettytable is not installed.
 ceph-common depends on python3-rados (= 16.2.10-1bionic); however:
  Package python3-rados is not installed.
 ceph-common depends on python3-rbd (= 16.2.10-1bionic); however:
  Package python3-rbd is not installed.
 ceph-common depends on python3-rgw (= 16.2.10-1bionic); however:
  Package python3-rgw is not installed.
 ceph-common depends on libaio1 (>= 0.3.93); however:
  Package libaio1 is not installed.
 ceph-common depends on libbabeltrace1 (>= 1.2.1); however:
  Package libbabeltrace1 is not installed.
 ceph-common depends on libcephfs2; however:
  Package libcephfs2 is not installed.
 ceph-common depends on libgoogle-perftools4; however:
  Package libgoogle-perftools4 is not installed.
 ceph-common depends on libleveldb1v5; however:
  Package libleveldb1v5 is not installed.
 ceph-common depends on liblua5.3-0; however:
  Package liblua5.3-0 is not installed.
 ceph-common depends on liboath0 (>= 1.10.0); however:
  Package liboath0 is not installed.
 ceph-common depends on librabbitmq4 (>= 0.8.0); however:
  Package librabbitmq4 is not installed.
 ceph-common depends on librados2; however:
  Package librados2 is not installed.
 ceph-common depends on libradosstriper1; however:
  Package libradosstriper1 is not installed.
 ceph-common depends on librdkafka1 (>= 0.9.2); however:
  Package librdkafka1 is not installed.
 ceph-common depends on libsnappy1v5; however:

```









# libaio1  

> depends on libaio1 (>= 0.3.93)

```bash
root@k8s-master01:/tmp/222# apt-cache madison libaio1
   libaio1 | 0.3.110-5ubuntu0.1 | http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 Packages
   libaio1 |  0.3.110-5 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/222#
root@k8s-master01:/tmp/222# apt install libaio1
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin libdw1
Use 'apt autoremove' to remove them.
The following NEW packages will be installed:
  libaio1
0 upgraded, 1 newly installed, 0 to remove and 47 not upgraded.
Need to get 6,476 B of archives.
After this operation, 30.7 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 libaio1 amd64 0.3.110-5ubuntu0.1 [6,476 B]
Fetched 6,476 B in 0s (124 kB/s)
Selecting previously unselected package libaio1:amd64.
(Reading database ... 68064 files and directories currently installed.)
Preparing to unpack .../libaio1_0.3.110-5ubuntu0.1_amd64.deb ...
Unpacking libaio1:amd64 (0.3.110-5ubuntu0.1) ...
Setting up libaio1:amd64 (0.3.110-5ubuntu0.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```



# libbabeltrace1

> depends on libbabeltrace1 (>= 1.2.1)

```b ash
root@k8s-master01:/tmp/222# apt-cache madison libbabeltrace1
libbabeltrace1 |    1.5.5-1 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages



```









```bash
root@k8s-master01:/tmp/222# apt-cache madison libgoogle-perftools4
libgoogle-perftools4 | 2.5-2.2ubuntu3 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/222#
root@k8s-master01:/tmp/222# apt install libgoogle-perftools4
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  libtcmalloc-minimal4
The following NEW packages will be installed:
  libgoogle-perftools4 libtcmalloc-minimal4
0 upgraded, 2 newly installed, 0 to remove and 47 not upgraded.
Need to get 281 kB of archives.
After this operation, 1,396 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mirrors.aliyun.com/ubuntu bionic/main amd64 libtcmalloc-minimal4 amd64 2.5-2.2ubuntu3 [91.6 kB]
Get:2 http://mirrors.aliyun.com/ubuntu bionic/main amd64 libgoogle-perftools4 amd64 2.5-2.2ubuntu3 [190 kB]
Fetched 281 kB in 0s (780 kB/s)
Selecting previously unselected package libtcmalloc-minimal4.
(Reading database ... 68086 files and directories currently installed.)
Preparing to unpack .../libtcmalloc-minimal4_2.5-2.2ubuntu3_amd64.deb ...
Unpacking libtcmalloc-minimal4 (2.5-2.2ubuntu3) ...
Selecting previously unselected package libgoogle-perftools4.
Preparing to unpack .../libgoogle-perftools4_2.5-2.2ubuntu3_amd64.deb ...
Unpacking libgoogle-perftools4 (2.5-2.2ubuntu3) ...
Setting up libtcmalloc-minimal4 (2.5-2.2ubuntu3) ...
Setting up libgoogle-perftools4 (2.5-2.2ubuntu3) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
```







```bash
root@k8s-master01:/tmp/222# apt-cache madison libleveldb1v5
libleveldb1v5 |     1.20-2 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/222#
root@k8s-master01:/tmp/222# apt install libleveldb1v5
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  libsnappy1v5
The following NEW packages will be installed:
  libleveldb1v5 libsnappy1v5
0 upgraded, 2 newly installed, 0 to remove and 47 not upgraded.
Need to get 152 kB of archives.
After this operation, 445 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mirrors.aliyun.com/ubuntu bionic/main amd64 libsnappy1v5 amd64 1.1.7-1 [16.0 kB]
Get:2 http://mirrors.aliyun.com/ubuntu bionic/main amd64 libleveldb1v5 amd64 1.20-2 [136 kB]
Fetched 152 kB in 0s (345 kB/s)
Selecting previously unselected package libsnappy1v5:amd64.
(Reading database ... 68104 files and directories currently installed.)
Preparing to unpack .../libsnappy1v5_1.1.7-1_amd64.deb ...
Unpacking libsnappy1v5:amd64 (1.1.7-1) ...
Selecting previously unselected package libleveldb1v5:amd64.
Preparing to unpack .../libleveldb1v5_1.20-2_amd64.deb ...
Unpacking libleveldb1v5:amd64 (1.20-2) ...
Setting up libsnappy1v5:amd64 (1.1.7-1) ...
Setting up libleveldb1v5:amd64 (1.20-2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```







```bash
root@k8s-master01:/tmp/222# apt-cache madison liblua5.3-0
liblua5.3-0 | 5.3.3-1ubuntu0.18.04.1 | http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 Packages
liblua5.3-0 | 5.3.3-1ubuntu0.18.04.1 | http://mirrors.aliyun.com/ubuntu bionic-security/main amd64 Packages
liblua5.3-0 |    5.3.3-1 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/222# apt install liblua5.3-0
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following NEW packages will be installed:
  liblua5.3-0
0 upgraded, 1 newly installed, 0 to remove and 47 not upgraded.
Need to get 115 kB of archives.
After this operation, 468 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 liblua5.3-0 amd64 5.3.3-1ubuntu0.18.04.1 [115 kB]
Fetched 115 kB in 0s (489 kB/s)
Selecting previously unselected package liblua5.3-0:amd64.
(Reading database ... 68114 files and directories currently installed.)
Preparing to unpack .../liblua5.3-0_5.3.3-1ubuntu0.18.04.1_amd64.deb ...
Unpacking liblua5.3-0:amd64 (5.3.3-1ubuntu0.18.04.1) ...
Setting up liblua5.3-0:amd64 (5.3.3-1ubuntu0.18.04.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```



```bash
root@k8s-master01:/tmp/222# apt-cache madison liboath0
  liboath0 |    2.6.1-1 | http://mirrors.aliyun.com/ubuntu bionic/universe amd64 Packages
root@k8s-master01:/tmp/222# apt install liboath0
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following NEW packages will be installed:
  liboath0
0 upgraded, 1 newly installed, 0 to remove and 47 not upgraded.
Need to get 44.7 kB of archives.
After this operation, 142 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic/universe amd64 liboath0 amd64 2.6.1-1 [44.7 kB]
Fetched 44.7 kB in 0s (238 kB/s)
Selecting previously unselected package liboath0.
(Reading database ... 68121 files and directories currently installed.)
Preparing to unpack .../liboath0_2.6.1-1_amd64.deb ...
Unpacking liboath0 (2.6.1-1) ...
Setting up liboath0 (2.6.1-1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```



```bash
root@k8s-master01:/tmp/222# apt-cache madison librabbitmq4
librabbitmq4 | 0.8.0-1ubuntu0.18.04.2 | http://mirrors.aliyun.com/ubuntu bionic-updates/universe amd64 Packages
librabbitmq4 | 0.8.0-1ubuntu0.18.04.2 | http://mirrors.aliyun.com/ubuntu bionic-security/universe amd64 Packages
librabbitmq4 | 0.8.0-1build1 | http://mirrors.aliyun.com/ubuntu bionic/universe amd64 Packages
root@k8s-master01:/tmp/222#
root@k8s-master01:/tmp/222# apt install librabbitmq4
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following NEW packages will be installed:
  librabbitmq4
0 upgraded, 1 newly installed, 0 to remove and 47 not upgraded.
Need to get 33.9 kB of archives.
After this operation, 104 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic-updates/universe amd64 librabbitmq4 amd64 0.8.0-1ubuntu0.18.04.2 [33.9 kB]
Fetched 33.9 kB in 0s (540 kB/s)
Selecting previously unselected package librabbitmq4:amd64.
(Reading database ... 68129 files and directories currently installed.)
Preparing to unpack .../librabbitmq4_0.8.0-1ubuntu0.18.04.2_amd64.deb ...
Unpacking librabbitmq4:amd64 (0.8.0-1ubuntu0.18.04.2) ...
Setting up librabbitmq4:amd64 (0.8.0-1ubuntu0.18.04.2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```







再次查看

```bash
root@k8s-master01:/tmp/222# dpkg -i ceph-common_16.2.10-1bionic_amd64.deb
Selecting previously unselected package ceph-common.
(Reading database ... 68142 files and directories currently installed.)
Preparing to unpack ceph-common_16.2.10-1bionic_amd64.deb ...
Unpacking ceph-common (16.2.10-1bionic) ...
dpkg: dependency problems prevent configuration of ceph-common:
 ceph-common depends on librbd1 (= 16.2.10-1bionic); however:
  Package librbd1 is not installed.
 ceph-common depends on python3-cephfs (= 16.2.10-1bionic); however:
  Package python3-cephfs is not installed.
 ceph-common depends on python3-ceph-argparse (= 16.2.10-1bionic); however:
  Package python3-ceph-argparse is not installed.
 ceph-common depends on python3-ceph-common (= 16.2.10-1bionic); however:
  Package python3-ceph-common is not installed.
 ceph-common depends on python3-prettytable; however:
  Package python3-prettytable is not installed.
 ceph-common depends on python3-rados (= 16.2.10-1bionic); however:
  Package python3-rados is not installed.
 ceph-common depends on python3-rbd (= 16.2.10-1bionic); however:
  Package python3-rbd is not installed.
 ceph-common depends on python3-rgw (= 16.2.10-1bionic); however:
  Package python3-rgw is not installed.
 ceph-common depends on libcephfs2; however:
  Package libcephfs2 is not installed.
 ceph-common depends on librados2; however:
  Package librados2 is not installed.
 ceph-common depends on libradosstriper1; however:
  Package libradosstriper1 is not installed.

dpkg: error processing package ceph-common (--install):
 dependency problems - leaving unconfigured
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Errors were encountered while processing:
 ceph-common

```



# librbd1_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i librbd1_16.2.10-1bionic_amd64.deb
(Reading database ... 68156 files and directories currently installed.)
Preparing to unpack librbd1_16.2.10-1bionic_amd64.deb ...
Unpacking librbd1 (16.2.10-1bionic) over (16.2.10-1bionic) ...
dpkg: dependency problems prevent configuration of librbd1:
 librbd1 depends on librados2 (= 16.2.10-1bionic); however:
  Package librados2 is not installed.
 librbd1 depends on liblttng-ust0 (>= 2.5.0); however:
  Package liblttng-ust0 is not installed.

dpkg: error processing package librbd1 (--install):
 dependency problems - leaving unconfigured
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
Errors were encountered while processing:
 librbd1

```



安装完librados2再次安装

```bash
root@k8s-master01:/tmp/223# dpkg -i librbd1_16.2.10-1bionic_amd64.deb
Selecting previously unselected package librbd1.
(Reading database ... 68254 files and directories currently installed.)
Preparing to unpack librbd1_16.2.10-1bionic_amd64.deb ...
Unpacking librbd1 (16.2.10-1bionic) ...
Setting up librbd1 (16.2.10-1bionic) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```







## librbd1_16.2.10-1bionic_amd64.deb > liblttng-ust0

```bash
root@k8s-master01:/tmp/223# apt install liblttng-ust0
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  liblttng-ust-ctl4 liburcu6
The following NEW packages will be installed:
  liblttng-ust-ctl4 liblttng-ust0 liburcu6
0 upgraded, 3 newly installed, 0 to remove and 47 not upgraded.
Need to get 287 kB of archives.
After this operation, 1,348 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mirrors.aliyun.com/ubuntu bionic/universe amd64 liblttng-ust-ctl4 amd64 2.10.1-1 [80.8 kB]
Get:2 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 liburcu6 amd64 0.10.1-1ubuntu1 [52.2 kB]
Get:3 http://mirrors.aliyun.com/ubuntu bionic/universe amd64 liblttng-ust0 amd64 2.10.1-1 [154 kB]
Fetched 287 kB in 1s (372 kB/s)
Selecting previously unselected package liblttng-ust-ctl4:amd64.
(Reading database ... 68142 files and directories currently installed.)
Preparing to unpack .../liblttng-ust-ctl4_2.10.1-1_amd64.deb ...
Unpacking liblttng-ust-ctl4:amd64 (2.10.1-1) ...
Selecting previously unselected package liburcu6:amd64.
Preparing to unpack .../liburcu6_0.10.1-1ubuntu1_amd64.deb ...
Unpacking liburcu6:amd64 (0.10.1-1ubuntu1) ...
Selecting previously unselected package liblttng-ust0:amd64.
Preparing to unpack .../liblttng-ust0_2.10.1-1_amd64.deb ...
Unpacking liblttng-ust0:amd64 (2.10.1-1) ...
Setting up liburcu6:amd64 (0.10.1-1ubuntu1) ...
Setting up liblttng-ust-ctl4:amd64 (2.10.1-1) ...
Setting up liblttng-ust0:amd64 (2.10.1-1) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
```



##  librbd1_16.2.10-1bionic_amd64.deb-> librados2

```bash
root@k8s-master01:/tmp/223# dpkg -i librados2_16.2.10-1bionic_amd64.deb
Selecting previously unselected package librados2.
(Reading database ... 68185 files and directories currently installed.)
Preparing to unpack librados2_16.2.10-1bionic_amd64.deb ...
Unpacking librados2 (16.2.10-1bionic) ...
dpkg: dependency problems prevent configuration of librados2:
 librados2 depends on libibverbs1 (>= 1.1.6); however:
  Package libibverbs1 is not installed.
 librados2 depends on librdmacm1 (>= 1.0.15); however:
  Package librdmacm1 is not installed.

dpkg: error processing package librados2 (--install):
 dependency problems - leaving unconfigured
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
Errors were encountered while processing:
 librados2

```



### librados2 -> libibverbs1

```bash
root@k8s-master01:/tmp/223# apt-cache madison libibverbs1
libibverbs1 | 17.1-1ubuntu0.2 | http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 Packages
libibverbs1 |     17.1-1 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/223# apt install libibverbs1
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  ibverbs-providers libnl-route-3-200
The following NEW packages will be installed:
  ibverbs-providers libibverbs1 libnl-route-3-200
0 upgraded, 3 newly installed, 0 to remove and 47 not upgraded.
Need to get 350 kB of archives.
After this operation, 1,282 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mirrors.aliyun.com/ubuntu bionic/main amd64 libnl-route-3-200 amd64 3.2.29-0ubuntu3 [146 kB]
Get:2 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 libibverbs1 amd64 17.1-1ubuntu0.2 [44.4 kB]
Get:3 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 ibverbs-providers amd64 17.1-1ubuntu0.2 [160 kB]
Fetched 350 kB in 0s (976 kB/s)
Selecting previously unselected package libnl-route-3-200:amd64.
(Reading database ... 68185 files and directories currently installed.)
Preparing to unpack .../libnl-route-3-200_3.2.29-0ubuntu3_amd64.deb ...
Unpacking libnl-route-3-200:amd64 (3.2.29-0ubuntu3) ...
Selecting previously unselected package libibverbs1:amd64.
Preparing to unpack .../libibverbs1_17.1-1ubuntu0.2_amd64.deb ...
Unpacking libibverbs1:amd64 (17.1-1ubuntu0.2) ...
Selecting previously unselected package ibverbs-providers:amd64.
Preparing to unpack .../ibverbs-providers_17.1-1ubuntu0.2_amd64.deb ...
Unpacking ibverbs-providers:amd64 (17.1-1ubuntu0.2) ...
Setting up libnl-route-3-200:amd64 (3.2.29-0ubuntu3) ...
Setting up libibverbs1:amd64 (17.1-1ubuntu0.2) ...
Setting up ibverbs-providers:amd64 (17.1-1ubuntu0.2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```



### librados2 -> librdmacm1

```bash
root@k8s-master01:/tmp/223# apt-cache madison librdmacm1
librdmacm1 | 17.1-1ubuntu0.2 | http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 Packages
librdmacm1 |     17.1-1 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/223# apt install librdmacm1
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following NEW packages will be installed:
  librdmacm1
0 upgraded, 1 newly installed, 0 to remove and 47 not upgraded.
Need to get 56.1 kB of archives.
After this operation, 169 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 librdmacm1 amd64 17.1-1ubuntu0.2 [56.1 kB]
Fetched 56.1 kB in 0s (834 kB/s)
Selecting previously unselected package librdmacm1:amd64.
(Reading database ... 68236 files and directories currently installed.)
Preparing to unpack .../librdmacm1_17.1-1ubuntu0.2_amd64.deb ...
Unpacking librdmacm1:amd64 (17.1-1ubuntu0.2) ...
Setting up librdmacm1:amd64 (17.1-1ubuntu0.2) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```





## 再次安装librados2

```bash
root@k8s-master01:/tmp/223# dpkg -i librados2_16.2.10-1bionic_amd64.deb
Selecting previously unselected package librados2.
(Reading database ... 68246 files and directories currently installed.)
Preparing to unpack librados2_16.2.10-1bionic_amd64.deb ...
Unpacking librados2 (16.2.10-1bionic) ...
Setting up librados2 (16.2.10-1bionic) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
```







# python3-cephfs_16.2.10-1bionic_amd64.deb

```b ash
root@k8s-master01:/tmp/223# dpkg -i python3-cephfs_16.2.10-1bionic_amd64.deb
Selecting previously unselected package python3-cephfs.
(Reading database ... 68267 files and directories currently installed.)
Preparing to unpack python3-cephfs_16.2.10-1bionic_amd64.deb ...
Unpacking python3-cephfs (16.2.10-1bionic) ...
dpkg: dependency problems prevent configuration of python3-cephfs:
 python3-cephfs depends on libcephfs2 (= 16.2.10-1bionic); however:
  Package libcephfs2 is not installed.
 python3-cephfs depends on python3-rados (= 16.2.10-1bionic); however:
  Package python3-rados is not installed.
 python3-cephfs depends on python3-ceph-argparse (= 16.2.10-1bionic); however:
  Package python3-ceph-argparse is not installed.

dpkg: error processing package python3-cephfs (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 python3-cephfs

```



## python3-cephfs_16.2.10-1bionic_amd64.deb >libcephfs2_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i libcephfs2_16.2.10-1bionic_amd64.deb
Selecting previously unselected package libcephfs2.
(Reading database ... 68267 files and directories currently installed.)
Preparing to unpack libcephfs2_16.2.10-1bionic_amd64.deb ...
Unpacking libcephfs2 (16.2.10-1bionic) ...
Setting up libcephfs2 (16.2.10-1bionic) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
```



## python3-cephfs_16.2.10-1bionic_amd64.deb >python3-rados_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-rados_16.2.10-1bionic_amd64.deb
Selecting previously unselected package python3-rados.
(Reading database ... 68271 files and directories currently installed.)
Preparing to unpack python3-rados_16.2.10-1bionic_amd64.deb ...
Unpacking python3-rados (16.2.10-1bionic) ...
Setting up python3-rados (16.2.10-1bionic) ...
```



## python3-cephfs_16.2.10-1bionic_amd64.deb >python3-ceph-argparse_16.2.10-1bionic_all.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-ceph-argparse_16.2.10-1bionic_all.deb
Selecting previously unselected package python3-ceph-argparse.
(Reading database ... 68278 files and directories currently installed.)
Preparing to unpack python3-ceph-argparse_16.2.10-1bionic_all.deb ...
Unpacking python3-ceph-argparse (16.2.10-1bionic) ...
Setting up python3-ceph-argparse (16.2.10-1bionic) ...

```





## 再次安装python3-cephfs_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-cephfs_16.2.10-1bionic_amd64.deb
Selecting previously unselected package python3-cephfs.
(Reading database ... 68282 files and directories currently installed.)
Preparing to unpack python3-cephfs_16.2.10-1bionic_amd64.deb ...
Unpacking python3-cephfs (16.2.10-1bionic) ...
Setting up python3-cephfs (16.2.10-1bionic) ...
```



## ceph-common > python3-ceph-argparse_16.2.10-1bionic_all.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-ceph-argparse_16.2.10-1bionic_all.deb
(Reading database ... 68290 files and directories currently installed.)
Preparing to unpack python3-ceph-argparse_16.2.10-1bionic_all.deb ...
Unpacking python3-ceph-argparse (16.2.10-1bionic) over (16.2.10-1bionic) ...
Setting up python3-ceph-argparse (16.2.10-1bionic) ...
```



## ceph-common >python3-ceph-common_16.2.10-1bionic_all.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-ceph-common_16.2.10-1bionic_all.deb
Selecting previously unselected package python3-ceph-common.
(Reading database ... 68290 files and directories currently installed.)
Preparing to unpack python3-ceph-common_16.2.10-1bionic_all.deb ...
Unpacking python3-ceph-common (16.2.10-1bionic) ...
Setting up python3-ceph-common (16.2.10-1bionic) ...
```





## ceph-common >python3-prettytable

```bash
root@k8s-master01:/tmp/223# apt-cache madison python3-prettytable
python3-prettytable |    0.7.2-3 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
root@k8s-master01:/tmp/223#
root@k8s-master01:/tmp/223# apt install python3-prettytable
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  docker-buildx-plugin docker-ce-rootless-extras docker-compose-plugin
Use 'apt autoremove' to remove them.
The following NEW packages will be installed:
  python3-prettytable
0 upgraded, 1 newly installed, 0 to remove and 54 not upgraded.
Need to get 19.7 kB of archives.
After this operation, 111 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic/main amd64 python3-prettytable all 0.7.2-3 [19.7 kB]
Fetched 19.7 kB in 0s (134 kB/s)
Selecting previously unselected package python3-prettytable.
(Reading database ... 68325 files and directories currently installed.)
Preparing to unpack .../python3-prettytable_0.7.2-3_all.deb ...
Unpacking python3-prettytable (0.7.2-3) ...
Setting up python3-prettytable (0.7.2-3) ...

```



## ceph-common > python3-rados_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-rados_16.2.10-1bionic_amd64.deb
(Reading database ... 68336 files and directories currently installed.)
Preparing to unpack python3-rados_16.2.10-1bionic_amd64.deb ...
Unpacking python3-rados (16.2.10-1bionic) over (16.2.10-1bionic) ...
Setting up python3-rados (16.2.10-1bionic) ...
```



## ceph-common > python3-rbd_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-rbd_16.2.10-1bionic_amd64.deb
Selecting previously unselected package python3-rbd.
(Reading database ... 68336 files and directories currently installed.)
Preparing to unpack python3-rbd_16.2.10-1bionic_amd64.deb ...
Unpacking python3-rbd (16.2.10-1bionic) ...
Setting up python3-rbd (16.2.10-1bionic) ...
root@k8s-master01:/tmp/223#
```



## ceph-common > python3-rgw_16.2.10-1bionic_amd64.deb> librgw2

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-rgw_16.2.10-1bionic_amd64.deb
Selecting previously unselected package python3-rgw.
(Reading database ... 68343 files and directories currently installed.)
Preparing to unpack python3-rgw_16.2.10-1bionic_amd64.deb ...
Unpacking python3-rgw (16.2.10-1bionic) ...
dpkg: dependency problems prevent configuration of python3-rgw:
 python3-rgw depends on librgw2 (>= 16.2.10-1bionic); however:
  Package librgw2 is not installed.


root@k8s-master01:/tmp/223# dpkg -i librgw2_16.2.10-1bionic_amd64.deb
Selecting previously unselected package librgw2.
(Reading database ... 68343 files and directories currently installed.)
Preparing to unpack librgw2_16.2.10-1bionic_amd64.deb ...
Unpacking librgw2 (16.2.10-1bionic) ...
Setting up librgw2 (16.2.10-1bionic) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

root@k8s-master01:/tmp/223# dpkg -i python3-rgw_16.2.10-1bionic_amd64.deb
(Reading database ... 68347 files and directories currently installed.)
Preparing to unpack python3-rgw_16.2.10-1bionic_amd64.deb ...
Unpacking python3-rgw (16.2.10-1bionic) over (16.2.10-1bionic) ...
Setting up python3-rgw (16.2.10-1bionic) ...

```





## ceph-common > python3-rbd_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i python3-rbd_16.2.10-1bionic_amd64.deb
Selecting previously unselected package python3-rbd.
(Reading database ... 68347 files and directories currently installed.)
Preparing to unpack python3-rbd_16.2.10-1bionic_amd64.deb ...
Unpacking python3-rbd (16.2.10-1bionic) ...
Setting up python3-rbd (16.2.10-1bionic) ...

```



## ceph-common >libradosstriper1_16.2.10-1bionic_amd64.deb

```bash
root@k8s-master01:/tmp/223# dpkg -i libradosstriper1_16.2.10-1bionic_amd64.deb
Selecting previously unselected package libradosstriper1.
(Reading database ... 68354 files and directories currently installed.)
Preparing to unpack libradosstriper1_16.2.10-1bionic_amd64.deb ...
Unpacking libradosstriper1 (16.2.10-1bionic) ...
Setting up libradosstriper1 (16.2.10-1bionic) ...
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...

```



# ceph-common

```bash
root@k8s-master01:/tmp/223# dpkg -i ceph-common_16.2.10-1bionic_amd64.deb
Selecting previously unselected package ceph-common.
(Reading database ... 68358 files and directories currently installed.)
Preparing to unpack ceph-common_16.2.10-1bionic_amd64.deb ...
Unpacking ceph-common (16.2.10-1bionic) ...
Setting up ceph-common (16.2.10-1bionic) ...
Adding group ceph....done
Adding system user ceph....done
Setting system user ceph properties....done
chown: cannot access '/var/log/ceph/*.log*': No such file or directory
Created symlink /etc/systemd/system/multi-user.target.wants/ceph.target → /lib/systemd/system/ceph.target.
Created symlink /etc/systemd/system/multi-user.target.wants/rbdmap.service → /lib/systemd/system/rbdmap.service.
Processing triggers for libc-bin (2.27-3ubuntu1.5) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...


root@k8s-master01:/tmp/223# ceph -v
ceph version 16.2.10 (45fa1a083152e41a408d15505f594ec5f1b4fe17) pacific (stable)
```

