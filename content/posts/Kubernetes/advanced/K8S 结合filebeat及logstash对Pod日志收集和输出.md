---
author: Ryan
title: K8S 结合filebeat及logstash对Pod日志收集和输出
date: 2023-12-07
lastmod: 2023-12-07
tags:
  - Kubernetes实战案例
  - 日志收集
categories:
  - Kubernetes
  - ELK
expirationReminder:
  enable: false
---

# K8S 结合filebeat及logstash对Pod日志收集和输出



## 一、安装Elasticsearch

### 1.1 安装软件包

```bash
root@es1:/tmp# dpkg -i elasticsearch-7.6.2-amd64.deb
Selecting previously unselected package elasticsearch.
(Reading database ... 67342 files and directories currently installed.)
Preparing to unpack elasticsearch-7.6.2-amd64.deb ...
Creating elasticsearch group... OK
Creating elasticsearch user... OK
Unpacking elasticsearch (7.6.2) ...
Setting up elasticsearch (7.6.2) ...
Created elasticsearch keystore in /etc/elasticsearch
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for systemd (237-3ubuntu10.52) ...

```

### 1.2 修改配置文件

```bash
 vim /etc/elasticsearch/elasticsearch.yml
#集群名称三个节点保持一致
cluster.name: xk8s-lgs
##   node身份不能一样
node.name: 192.168.149.22
## 数据路径
path.data: /data/elasticsearch
path.logs: /data/log/elasticsearch
## 监听地址也可以设置为0.0.0.0
network.host: 192.168.149.22
http.port: 9200
## 集群发现节点的配置有几个节点写几个
discovery.seed_hosts: ["192.168.149.22", "192.168.149.23","192.168.149.24"]
## 那些节点可以被选为master
cluster.initial_master_nodes: ["192.168.149.22", "192.168.149.23","192.168.149.24"]
##这个设置表示在集群中至少有指定数量的节点启动并加入后，Elasticsearch 集群才会开始执行数据恢复操作
gateway.recover_after_nodes: 2

##这个设置强制要求在执行例如删除索引、删除模板等时必须提供名称
action.destructive_requires_name: true
```



### 1.3 创建数据路径和权限

```bash
mkdir -p /data/log/elasticsearch && mkdir -p /data/elasticsearch
chown -R elasticsearch.elasticsearch  /data/elasticsearch/ && chown -R elasticsearch.elasticsearch  /data/log/elasticsearch/
```



### 1.4 启动服务

```bash
root@es1:/data/log#  systemctl enable elasticsearch.service --now
Synchronizing state of elasticsearch.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable elasticsearch
Created symlink /etc/systemd/system/multi-user.target.wants/elasticsearch.service → /usr/lib/systemd/system/elasticsearch.service.
```



### 1.5 检查运行状态

```bash
root@es1:/data/log# systemctl status elasticsearch.service
● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2023-12-07 02:10:29 UTC; 12s ago
     Docs: http://www.elastic.co
 Main PID: 21935 (java)
    Tasks: 45 (limit: 4629)
   CGroup: /system.slice/elasticsearch.service
           ├─21935 /usr/share/elasticsearch/jdk/bin/java -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss
           └─22054 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

Dec 07 02:10:14 es1 systemd[1]: Starting Elasticsearch...
Dec 07 02:10:15 es1 elasticsearch[21935]: OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be remov
Dec 07 02:10:29 es1 systemd[1]: Started Elasticsearch.
```



```bash
## 查看服务端口
root@es1:/data/log# ss -tnl
State             Recv-Q             Send-Q                                      Local Address:Port                           Peer Address:Port
LISTEN            0                  128                                         127.0.0.53%lo:53                                  0.0.0.0:*
LISTEN            0                  128                                               0.0.0.0:22                                  0.0.0.0:*
LISTEN            0                  128                                             127.0.0.1:6010                                0.0.0.0:*
LISTEN            0                  128                                             127.0.0.1:6011                                0.0.0.0:*
LISTEN            0                  128                               [::ffff:192.168.149.22]:9200                                      *:*
LISTEN            0                  128                               [::ffff:192.168.149.22]:9300                                      *:*
```





## 二、安装zookeeper

### 2.1 安装openjdk-8

```
apt update

apt install openjdk-8-jdk -y
```



### 2.2 安装zookeeper

```bash
root@zk1:~# mkdir /apps
root@zk1:~# mv /tmp/apache-zookeeper-3.5.9-bin.tar.gz /apps/
root@zk1:~# tar -xf /apps/apache-zookeeper-3.5.9-bin.tar.gz

root@zk1:~# scp /apps/apache-zookeeper-3.5.9-bin.tar.gz xin@192.168.149.27:/tmp
The authenticity of host '192.168.149.27 (192.168.149.27)' can't be established.
ECDSA key fingerprint is SHA256:A8lG3v6uOS8zPSzTlx/kLoRBhgMFUK9I5pngntoeVWc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.149.27' (ECDSA) to the list of known hosts.
xin@192.168.149.27's password:
apache-zookeeper-3.5.9-bin.tar.gz                                                                                         100% 9397KB  91.3MB/s   00:00
root@zk1:~# scp /apps/apache-zookeeper-3.5.9-bin.tar.gz xin@192.168.149.28:/tmp
The authenticity of host '192.168.149.28 (192.168.149.28)' can't be established.
ECDSA key fingerprint is SHA256:A8lG3v6uOS8zPSzTlx/kLoRBhgMFUK9I5pngntoeVWc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.149.28' (ECDSA) to the list of known hosts.
xin@192.168.149.28's password:
apache-zookeeper-3.5.9-bin.tar.gz                                                                                         100% 9397KB 135.1MB/s   00:00
```



```bash
root@zk1:/apps# tar xf apache-zookeeper-3.5.9-bin.tar.gz
root@zk1:/apps# cd apache-zookeeper-3.5.9-bin/
root@zk1:/apps/apache-zookeeper-3.5.9-bin# ls
LICENSE.txt  NOTICE.txt  README.md  README_packaging.txt  bin  conf  docs  lib
root@zk1:/apps/apache-zookeeper-3.5.9-bin# cd conf/
root@zk1:/apps/apache-zookeeper-3.5.9-bin/conf# ls
configuration.xsl  log4j.properties  zoo_sample.cfg
root@zk1:/apps/apache-zookeeper-3.5.9-bin/conf# cp zoo_sample.cfg zoo.cfg

mkdir -p /data/zookeeper
```



### 2.3 编辑zookeeper配置文件

```bash
root@zk1:/apps/apache-zookeeper-3.5.9-bin/conf# vim zoo.cfg

# tickTime 每个 tick 的时间长度，单位为毫秒。ZooKeeper 使用 tick 作为时间单位来衡量时间。
tickTime=2000

# 初始同步阶段可以持续的 tick 数。这个设置指定了 ZooKeeper 集群启动时，follower 节点与 leader 节点进行初始同步的最长时间。在这个阶段内，follower 节点将尽力与 leader 进行同步。
initLimit=10
# 发送请求和收到确认之间允许的最大 tick 数。在这个设置的时间内，如果一个 follower 节点发送请求但未收到确认，则认为同步失败。
syncLimit=5
#存储 ZooKeeper 快照的目录。ZooKeeper 保存的数据包括事务日志和快照，这个目录用于存放快照数据。
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181


# 最大客户端连接数。这个设置限制了能够连接到 ZooKeeper 服务器的最大客户端连接数。
maxClientCnxns=60

#目录中要保留的快照数量。ZooKeeper 会定期创建快照来备份数据，并根据这个设置保留最近的几个快照。
autopurge.snapRetainCount=3

#定义ZooKeeper 集群中各个节点的信息，包括节点的 ID、IP 地址以及用于通信的端口号。
server.1=192.168.149.26:2888:3888
server.2=192.168.149.27:2888:3888
server.3=192.168.149.28:2888:3888
```



### 2.4 配置zookeeper

```bash
root@zk1:/apps/apache-zookeeper-3.5.9-bin/conf# scp zoo.cfg xin@192.168.149.27:/tmp
root@zk1:/apps/apache-zookeeper-3.5.9-bin/conf# scp zoo.cfg xin@192.168.149.28:/tmp     

root@zk1:/apps/apache-zookeeper-3.5.9-bin/conf# echo 1 > /data/zookeeper/myid
root@zk2:/apps/apache-zookeeper-3.5.9-bin/conf# echo 2 > /data/zookeeper/myid
root@zk3:/apps/apache-zookeeper-3.5.9-bin/conf# echo 3 > /data/zookeeper/myid
```



```bash
## 创建软连接
ln -s /apps/apache-zookeeper-3.5.9-bin /apps/zookeeper
```



### 2.5 使用Systemd 管理zookeeper

```bash
vim /etc/systemd/system/zookeeper.service
[Unit]
Description=Zookeeper Service unit Configuration
After=network.target

[Service]
Type=forking
ExecStart=/apps/zookeeper/bin/zkServer.sh start /apps/zookeeper/conf/zoo.cfg
ExecStop=/apps/zookeeper/bin/zkServer.sh stop
PIDFile=/data/zookeeper/zookeeper_server.pid
KillMode=none
User=root
Group=root
Restart=on-failure
[Install]
WantedBy=multi-user.target
```





### 2.5 启动服务

```bash
root@zk1:/data/zookeeper# systemctl status zookeeper.service
● zookeeper.service - Zookeeper Service unit Configuration
   Loaded: loaded (/etc/systemd/system/zookeeper.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2023-12-07 08:18:43 UTC; 32s ago
  Process: 104123 ExecStart=/apps/zookeeper/bin/zkServer.sh start /apps/zookeeper/conf/zoo.cfg (code=exited, status=0/SUCCESS)
 Main PID: 104151 (java)
    Tasks: 39 (limit: 2287)
   CGroup: /system.slice/zookeeper.service
           └─104151 java -Dzookeeper.log.dir=/apps/zookeeper/bin/../logs -Dzookeeper.log.file=zookeeper-root-server-zk1.log -Dzookeeper.root.logger=INFO,CON

Dec 07 08:18:42 zk1 systemd[1]: Starting Zookeeper Service unit Configuration...
Dec 07 08:18:42 zk1 zkServer.sh[104123]: /usr/bin/java
Dec 07 08:18:42 zk1 zkServer.sh[104123]: ZooKeeper JMX enabled by default
Dec 07 08:18:42 zk1 zkServer.sh[104123]: Using config: /apps/zookeeper/conf/zoo.cfg
Dec 07 08:18:43 zk1 zkServer.sh[104123]: Starting zookeeper ... STARTED
Dec 07 08:18:43 zk1 systemd[1]: Started Zookeeper Service unit Configuration.
lines 1-15/15 (END)
root@zk1:/data/zookeeper#
root@zk1:/data/zookeeper#
root@zk1:/data/zookeeper# ss -tnl
State             Recv-Q             Send-Q                                     Local Address:Port                            Peer Address:Port
LISTEN            0                  128                                            127.0.0.1:6010                                 0.0.0.0:*
LISTEN            0                  128                                            127.0.0.1:6011                                 0.0.0.0:*
LISTEN            0                  128                                            127.0.0.1:6012                                 0.0.0.0:*
LISTEN            0                  128                                        127.0.0.53%lo:53                                   0.0.0.0:*
LISTEN            0                  128                                              0.0.0.0:22                                   0.0.0.0:*
LISTEN            0                  128                                                [::1]:6010                                    [::]:*
LISTEN            0                  128                                                [::1]:6011                                    [::]:*
LISTEN            0                  128                                                [::1]:6012                                    [::]:*
LISTEN            0                  50                                                     *:36673                                      *:*
LISTEN            0                  50                                                     *:2181                                       *:*

```



### 2.6 查看服务状态

```bash
root@zk1:/data/zookeeper/version-2# /apps/zookeeper/bin/zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /apps/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower

root@zk2:/data/zookeeper/version-2# /apps/zookeeper/bin/zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /apps/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader

root@zk3:/data/zookeeper/version-2# /apps/zookeeper/bin/zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /apps/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
```

