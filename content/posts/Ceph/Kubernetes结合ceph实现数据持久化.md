



rbd结合K8s提供存储卷及动态卷使用案例

让Kubernetes中的Pod能够访问Ceph中的RBD（块设备镜像）作为存储设备，需要在ceph创建rbd并且让k8s node 节点能够通过ceph认证。

> 在使用Ceph作为动态存储卷的时候，确实需要确保Kubernetes集群中的各个组件，包括kube-controller-manager，能够访问Ceph存储集群。为了实现这一点，通常需要在每个Kubernetes节点上同步Ceph的认证文件。

可以根据需求在Kubernetes中选择使用RBD存储或CephFS存储。

RBD（Rados Block Device）通常用于有状态服务，因为它提供了块设备级别的存储，适用于需要持久性和存储卷的应用程序，如数据库或其他有状态服务。RBD卷可以附加到Pod，并且可以在Pod重新调度时保留数据。这使其非常适用于需要持久性数据的应用程序。

CephFS（Ceph文件系统）通常用于无状态服务，因为它提供了一个分布式文件系统，允许多个Pod在同一文件系统上共享数据。这对于需要跨多个Pod之间共享数据的应用程序非常有用，如Web服务器或应用程序日志。



## Kubernetes 通过keyring文件挂载ceph rbd 镜像



ceph集群状态







```bash
ceph health detail | awk '{print $2}' | sed -n '/^2\.*/p' | sed -n '1,50p'
2.3e 2.3f 2.3c 2.3d 2.3a 2.3b 2.38 2.39 2.36 2.37 2.34 2.35 2.32 2.33 2.30 2.31 2.2e 2.2f 2.2c 2.2d 2.2a 2.2b 2.28 2.29 2.26 2.27 2.24
2.25 2.22 2.23 2.20 2.21 2.1b 2.1a 2.19 2.18 2.17 2.16 2.15 2.14 2.13 2.12 2.11 2.10 2.f 2.e 2.d 2.c 2.5 2.6
```



```bash
#!/bin/bash

for i in $(ceph health detail | awk '{print $2}' | sed -n '/^2\.*/p' | sed -n '1,50p'); do
    ceph pg deep-scrub $i
done

```

```bash
root@ceph-mon1[10:16:36]~ #:sh deep-scrub.sh
instructing pg 2.3e on osd.12 to deep-scrub
instructing pg 2.3f on osd.7 to deep-scrub
instructing pg 2.3c on osd.7 to deep-scrub
instructing pg 2.3d on osd.1 to deep-scrub
instructing pg 2.3a on osd.12 to deep-scrub
instructing pg 2.3b on osd.8 to deep-scrub
instructing pg 2.38 on osd.12 to deep-scrub
instructing pg 2.39 on osd.14 to deep-scrub
instructing pg 2.36 on osd.1 to deep-scrub
instructing pg 2.37 on osd.13 to deep-scrub
instructing pg 2.34 on osd.5 to deep-scrub
instructing pg 2.35 on osd.1 to deep-scrub
instructing pg 2.32 on osd.12 to deep-scrub
instructing pg 2.33 on osd.8 to deep-scrub
instructing pg 2.30 on osd.13 to deep-scrub
instructing pg 2.31 on osd.0 to deep-scrub
instructing pg 2.2e on osd.7 to deep-scrub
instructing pg 2.2f on osd.10 to deep-scrub
instructing pg 2.2c on osd.14 to deep-scrub
instructing pg 2.2d on osd.8 to deep-scrub
instructing pg 2.2a on osd.9 to deep-scrub
instructing pg 2.2b on osd.4 to deep-scrub
instructing pg 2.28 on osd.3 to deep-scrub
instructing pg 2.29 on osd.6 to deep-scrub
instructing pg 2.26 on osd.9 to deep-scrub
instructing pg 2.27 on osd.9 to deep-scrub
instructing pg 2.24 on osd.6 to deep-scrub
instructing pg 2.25 on osd.13 to deep-scrub
instructing pg 2.22 on osd.6 to deep-scrub
instructing pg 2.23 on osd.4 to deep-scrub
instructing pg 2.20 on osd.7 to deep-scrub
instructing pg 2.21 on osd.4 to deep-scrub
instructing pg 2.1b on osd.11 to deep-scrub
instructing pg 2.1a on osd.6 to deep-scrub
instructing pg 2.19 on osd.3 to deep-scrub
instructing pg 2.18 on osd.0 to deep-scrub
instructing pg 2.17 on osd.6 to deep-scrub
instructing pg 2.16 on osd.8 to deep-scrub
instructing pg 2.15 on osd.10 to deep-scrub
instructing pg 2.14 on osd.8 to deep-scrub
instructing pg 2.13 on osd.13 to deep-scrub
instructing pg 2.12 on osd.10 to deep-scrub
instructing pg 2.11 on osd.6 to deep-scrub
instructing pg 2.10 on osd.10 to deep-scrub
instructing pg 2.f on osd.11 to deep-scrub
instructing pg 2.e on osd.2 to deep-scrub
instructing pg 2.d on osd.12 to deep-scrub
instructing pg 2.c on osd.12 to deep-scrub
instructing pg 2.5 on osd.12 to deep-scrub
instructing pg 2.6 on osd.1 to deep-scrub
```



再次检查

```bash
root@ceph-mon1[10:24:19]~ #:ceph health detail
HEALTH_OK
```







查看存储池

```bash
xceo@ceph-mon1:~$ ceph osd pool ls
device_health_metrics
xxrbd3
xxrbd2
cephfs-metadata
cephfs-data
.rgw.root
default.rgw.log
default.rgw.control
default.rgw.meta
```



查看存储池使用情况

```bash
xceo@ceph-mon1:~$ ceph df
--- RAW STORAGE ---
CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
hdd    1.8 TiB  1.2 TiB  562 GiB   562 GiB      31.23
TOTAL  1.8 TiB  1.2 TiB  562 GiB   562 GiB      31.23

--- POOLS ---
POOL                   ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
device_health_metrics   1    1      0 B        0      0 B      0    322 GiB
xxrbd3                  3   64   10 MiB       17   31 MiB      0    322 GiB
xxrbd2                  4   64   12 MiB       16   37 MiB      0    322 GiB
cephfs-metadata         5   32  226 MiB      106  679 MiB   0.07    322 GiB
cephfs-data             6   64  183 GiB   47.35k  550 GiB  36.30    322 GiB
.rgw.root               7   32  1.3 KiB        4   48 KiB      0    322 GiB
default.rgw.log         8   32  3.6 KiB      209  408 KiB      0    322 GiB
default.rgw.control     9   32      0 B        8      0 B      0    322 GiB
default.rgw.meta       10   32      0 B        0      0 B      0    322 GiB
```



创建存储池

```bash
## 创建存储池 k8s-xrbd-pool1
xceo@ceph-mon1:~$ ceph osd pool create k8s-xrbd-pool1 32 32
pool 'k8s-xrbd-pool1' created

## 验证存储池
xceo@ceph-mon1:~$ ceph osd pool ls
k8s-xrbd-pool1

## 存储池启用rbd块存储功能
xceo@ceph-mon1:~$ ceph osd pool application enable k8s-xrbd-pool1 rbd
enabled application 'rbd' on pool 'k8s-xrbd-pool1'

## 初始化rbd
xceo@ceph-mon1:~$ rbd pool init -p k8s-xrbd-pool1
```



创建Image

创建好的rbd不能直接挂载需要创建镜像

```bash
## 创建镜像
xceo@ceph-mon1:~$ rbd create k8s-xrbd-img1 --size 4G --pool k8s-xrbd-pool1 --image-feature layering

## 查看镜像
xceo@ceph-mon1:~$ rbd ls --pool k8s-xrbd-pool1
k8s-xrbd-img1

##验证镜像信息
xceo@ceph-mon1:~$ rbd --image k8s-xrbd-img1 --pool k8s-xrbd-pool1 info
rbd image 'k8s-xrbd-img1':
        size 4 GiB in 1024 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: 12bcae2102c6
        block_name_prefix: rbd_data.12bcae2102c6
        format: 2
        features: layering
        op_features:
        flags:
        create_timestamp: Tue Oct 31 10:44:52 2023
        access_timestamp: Tue Oct 31 10:44:52 2023
        modify_timestamp: Tue Oct 31 10:44:52 2023
```



### 客户端安装ceph-common

分别在K8S Master与各node 节点安装 ceph-common 组件包。

```BASH
## 查看当前主机系统版本
root@k8s-master01:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.6 LTS
Release:        18.04
Codename:       bionic
```



```bash
## 安装key
$ wget -q -O- 'https://mirrors.aliyun.com/ceph/keys/release.asc' | sudo apt-key add -

## Ceph pacific 版本
$ sudo apt-add-repository 'deb https://mirrors.aliyun.com/ceph/debian-pacific/ bionic main'
$ sudo apt update

## 查看版

```





## Kubernetes 通secret 挂载rbd及cephfs 使用案例



