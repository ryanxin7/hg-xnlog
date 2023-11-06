# Kubernetes结合ceph实现数据持久化


# Ceph K8s环境rdb,CephFS的使用

让Kubernetes中的Pod能够访问Ceph中的RBD（块设备镜像）作为存储设备，需要在ceph创建rbd并且让k8s node 节点能够通过ceph认证。

> 在使用Ceph作为动态存储卷的时候，确实需要确保Kubernetes集群中的各个组件，包括kube-controller-manager，能够访问Ceph存储集群。为了实现这一点，通常需要在每个Kubernetes节点上同步Ceph的认证文件。

可以根据需求在Kubernetes中选择使用RBD存储或CephFS存储。

RBD（Rados Block Device）通常用于有状态服务，因为它提供了块设备级别的存储，适用于需要持久性和存储卷的应用程序，如数据库或其他有状态服务。RBD卷可以附加到Pod，并且可以在Pod重新调度时保留数据。这使其非常适用于需要持久性数据的应用程序。

CephFS（Ceph文件系统）通常用于无状态服务，因为它提供了一个分布式文件系统，允许多个Pod在同一文件系统上共享数据。这对于需要跨多个Pod之间共享数据的应用程序非常有用，如Web服务器或应用程序日志。



## 1.1 基于rbd结合k8s提供存储卷及动态存储



**查看ceph集群状态**

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



### 1.1.1 创建存储池

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



### 1.1.2 创建Image

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



### 1.1.3 客户端安装ceph-common

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
wget -q -O- 'https://mirrors.aliyun.com/ceph/keys/release.asc' | sudo apt-key add -

## Ceph pacific 版本
sudo apt-add-repository 'deb https://mirrors.aliyun.com/ceph/debian-pacific/ bionic main'
sudo apt update

## 查看软件包版本
apt-cache madison ceph-common
ceph-common | 16.2.14-1bionic | https://mirrors.aliyun.com/ceph/debian-pacific bionic/main amd64 Packages
ceph-common | 15.2.17-1bionic | https://mirrors.aliyun.com/ceph/debian-octopus bionic/main amd64 Packages
ceph-common | 12.2.13-0ubuntu0.18.04.11 | http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 Packages
ceph-common | 12.2.13-0ubuntu0.18.04.11 | http://mirrors.aliyun.com/ubuntu bionic-security/main amd64 Packages
ceph-common | 12.2.4-0ubuntu1 | http://mirrors.aliyun.com/ubuntu bionic/main amd64 Packages
```



因为Ceph集群的版本为16.2.10，Common 的版本尽量和Ceph集群的版本一致，软件包一般只提供最新的版本指定版本的话需要手动下载deb文件进行安装。

16.2.10 版本DEB软件包下载：https://mirrors.aliyun.com/ceph/debian-16.2.10/pool/main/c/ceph/?spm=a2c6h.25603864.0.0.27912add02vFGg



**遇到依赖问题**

解决方法：[依赖问题解决](ceph-common-依赖问题)

```bash
 $ dpkg -i ceph-common_16.2.10-1bionic_amd64.deb
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





```bash
## 安装依赖
apt install -y libaio1 libbabeltrace1 libgoogle-perftools4 libleveldb1v5 liblua5.3-0 liboath0 librabbitmq4 liblttng-ust0 librdmacm1 libibverbs1 librdkafka1 python3-prettytable 
```



```bash
## 解压安装包
tar -xf /tmp/Ceph-Common-16.2.10.tar -C /opt/ceph
```



```bash
## 执行安装脚本
cd /opt/ceph/Ceph-Common-16.2.10
bash ./install-ceph-common-16.2.10.sh 
...
Successfully installed ceph-common_16.2.10-1bionic_amd64.deb
Ceph packages installation completed.
```



```bash
## 验证版本
root@k8s-node01:~# ceph -v
ceph version 16.2.10 (45fa1a083152e41a408d15505f594ec5f1b4fe17) pacific (stable)

root@k8s-node02:~# ceph -v
ceph version 16.2.10 (45fa1a083152e41a408d15505f594ec5f1b4fe17) pacific (stable)
```



### 1.1.4 创建Ceph普通用户权限keyring

```bash
xceo@ceph-mon1:~/ceph-cluster$ ceph auth get-or-create client.admk8s-ceamg mon 'allow r' osd 'allow * pool=k8s-xrbd-pool1'
[client.admk8s-ceamg]
        key = AQAhr0hlhZTGCxAAajU0BbOfxO2+oUJ8OkmnXA==
```



```bash
## 验证用户
xceo@ceph-mon1:~/ceph-cluster$ ceph auth get client.admk8s-ceamg
[client.admk8s-ceamg]
        key = AQAhr0hlhZTGCxAAajU0BbOfxO2+oUJ8OkmnXA==
        caps mon = "allow r"
        caps osd = "allow * pool=k8s-xrbd-pool1"
exported keyring for client.admk8s-ceamg
```



```bash 
## 导出用户信息到Keying文件
xceo@ceph-mon1:~/ceph-cluster$ ceph auth get client.admk8s-ceamg -o ceph.client.admk8s-ceamg.keyring
exported keyring for client.admk8s-ceamg

xceo@ceph-mon1:~/ceph-cluster$ cat ceph.client.admk8s-ceamg.keyring
[client.admk8s-ceamg]
        key = AQAhr0hlhZTGCxAAajU0BbOfxO2+oUJ8OkmnXA==
        caps mon = "allow r"
        caps osd = "allow * pool=k8s-xrbd-pool1"

```

```bash
## 同步认证文件到K8s 各master和node节点
xceo@ceph-mon1:~/ceph-cluster$ scp ceph.client.admk8s-ceamg.keyring ceph.conf root@10.1.0.110:/etc/ceph
xceo@ceph-mon1:~/ceph-cluster$ scp ceph.client.admk8s-ceamg.keyring ceph.conf root@10.1.0.111:/etc/ceph
xceo@ceph-mon1:~/ceph-cluster$ scp ceph.client.admk8s-ceamg.keyring ceph.conf root@10.1.0.112:/etc/ceph
```





```bash
## hosts
10.1.0.39 ceph-node1.xx.local ceph-node1
10.1.0.40 ceph-node2.xx.local ceph-node2
10.1.0.41 ceph-node3.xx.local ceph-node3
10.1.0.39 ceph-mon1.xx.local ceph-mon1
10.1.0.40 ceph-mon2.xx.local ceph-mon2
10.1.0.41 ceph-mon3.xx.local ceph-mon3
10.1.0.40 ceph-mgr1.xx.local ceph-mgr1
10.1.0.41 ceph-mgr2.xx.local ceph-mgr2
10.1.0.39 ceph-deploy.xx.local ceph-deploy
```



```bash 
## 在k8s node 节点验证用户权限
root@k8s-node01:/etc/ceph# ceph --user admk8s-ceamg.xx -s
  cluster:
    id:     62be32df-9cb4-474f-8727-d5c4bbceaf97
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph-mon1,ceph-mon2,ceph-mon3 (age 11h)
    mgr: ceph-mgr1(active, since 5M), standbys: ceph-mgr2
    mds: 1/1 daemons up
    osd: 15 osds: 15 up (since 5M), 15 in (since 5M)
    rgw: 1 daemon active (1 hosts, 1 zones)

  data:
    volumes: 1/1 healthy
    pools:   10 pools, 385 pgs
    objects: 47.71k objects, 184 GiB
    usage:   562 GiB used, 1.2 TiB / 1.8 TiB avail
    pgs:     385 active+clean
```





```bash
## 验证镜像访问权限
root@k8s-node02:/etc/ceph# rbd --id admk8s-ceamg.xx ls --pool=k8s-xrbd-pool1
k8s-xrbd-img1
```



## 1.2  通过Keyring 文件挂载RBD

 k8s环境可以有2种方式挂载rbd.

1. 基于keyring
2. 基于k8s secret



### 1.2.1 通过Kering文件直接挂载-busybox

```yaml
# cat case1-busybox-keyring.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox 
    command:
      - sleep
      - "3600"
    imagePullPolicy: Always 
    name: busybox
    #restartPolicy: Always
    volumeMounts:
    - name: rbd-data1
      mountPath: /data
  volumes:
    - name: rbd-data1
      rbd:
        monitors:
        - '10.1.0.39:6789'
        - '10.1.0.40:6789'
        - '10.1.0.41:6789'
        pool: k8s-xrbd-pool1
        image: k8s-xrbd-img1
        fsType: ext4
        readOnly: false
        user: admk8s-ceamg
        keyring: /etc/ceph/ceph.client.admk8s-ceamg.keyring


## 部署busybox
# kubectl apply -f case1-busybox-keyring.yaml

Events:
  Type    Reason                  Age   From                     Message
  ----    ------                  ----  ----                     -------
  Normal  Scheduled               14s   default-scheduler        Successfully assigned default/busybox to k8s-node03
  Normal  SuccessfulAttachVolume  15s   attachdetach-controller  AttachVolume.Attach succeeded for volume "rbd-data1"
  Normal  Pulling                 4s    kubelet                  Pulling image "busybox:1.34"
  Normal  Pulled                  3s    kubelet                  Successfully pulled image "busybox:1.34" in 1.841793996s
  Normal  Created                 3s    kubelet                  Created container busybox
  Normal  Started                 2s    kubelet                  Started container busybox


root@k8s-master01:/etc/ceph# kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          16s



## 此时busybox已经启动
root@k8s-master01:/etc/ceph# kubectl get pod -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          52s   10.244.1.4   k8s-node03   <none>           <none>
root@k8s-master01:/etc/ceph#
```







## Kubernetes 通secret 挂载rbd及cephfs 使用案例





---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/ceph/kubernetes%E7%BB%93%E5%90%88ceph%E5%AE%9E%E7%8E%B0%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E5%8C%96/  

