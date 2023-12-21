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







## 三、安装kafka

### 3.1  下载 kafka程序包安装

https://kafka.apache.org/downloads



```bash
tar xf kafka_2.13-2.4.1.tgz
mkdir /data/kafka-logs
ln -s kafka_2.13-2.4.1  kafka
vim config/server.properties
```





### 3.2 编辑kafka配置文件

```bash


```









```bash
root@zk1:/apps/kafka# systemctl status kafka.service
● kafka.service - Apache Kafka server (broker)
   Loaded: loaded (/etc/systemd/system/kafka.service; disabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-12-08 02:15:50 UTC; 5s ago
 Main PID: 94927 (java)
    Tasks: 23 (limit: 2287)
   CGroup: /system.slice/kafka.service
           └─94927 java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -D

Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,194] INFO Client environment:user.home=/root (org.apache.zookeeper.ZooKeeper)
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,195] INFO Client environment:user.dir=/ (org.apache.zookeeper.ZooKeeper)
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,195] INFO Client environment:os.memory.free=973MB (org.apache.zookeeper.ZooKeeper)
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,196] INFO Client environment:os.memory.max=1024MB (org.apache.zookeeper.ZooKeeper)
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,196] INFO Client environment:os.memory.total=1024MB (org.apache.zookeeper.ZooKeeper)
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,203] INFO Initiating client connection, connectString=192.167.149.26:2181,192.168.149
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,304] INFO Setting -D jdk.tls.rejectClientInitiatedRenegotiation=true to disable clien
Dec 08 02:15:53 zk1 kafka-server-start.sh[94927]: [2023-12-08 02:15:53,324] INFO jute.maxbuffer value is 4194304 Bytes (org.apache.zookeeper.ClientCnxnSocke
```



```bash
root@zk3:/apps# systemctl enable kafka.service
Created symlink /etc/systemd/system/multi-user.target.wants/kafka.service → /etc/systemd/system/kafka.service.
```



![image-20231208104323045](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231208104323045.png)

![image-20231208104413087](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231208104413087.png)



## 四、在业务镜像中加入filebeat



### 4.1 centos dockerfile

```docker
FROM centos:7.8.2003
LABEL maintainer="Ryan"

# 添加文件到镜像中
ADD filebeat-7.6.2-x86_64.rpm /tmp/

# 安装所需软件包
RUN yum install -y vim wget tree lrzsz gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop && \
    rm -rf /var/cache/yum/* && \
    yum clean all

# 设置系统时区为上海时区
RUN rm -rf /etc/localtime && \
    ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 添加用户到系统中
RUN useradd www -u 2023 && \
    useradd nginx -u 2024
```



### 4.2 JDK dockerfile

```dockerfile
FROM harbor.ceamg.com/baseimages/centos7.8-filebate:x1
MAINTAINER xinn.cc
WORKDIR /usr/local/java17
ADD ./jdk-17.0.9_linux-x64.tar.gz  /usr/local/java17/
ENV JAVA_HOME=/usr/local/java17/jdk-17.0.9
ENV CLASSPATH=.:$JAVA_HOME/lib/jrt-fs.jar
ENV PATH=$PATH:$JAVA_HOME/bin
```



`harbor.ceamg.com/baseimages/jdk17_0.9:x3`



### 4.3 tomcat dockerfile

```dockerfile
FROM harbor.ceamg.com/baseimages/jdk17_0.9:x3

# Tomcat version to be installed
ENV TOMCAT_VERSION 10.1.16

COPY ./apache-tomcat-10.1.16.tar.gz /tmp/

# 在容器中解压缩Tomcat文件
RUN tar -xf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz -C /opt/ && \
    mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \
    rm -rf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz

ADD filebeat.yaml /etc/filebeat/filebeat.yaml
ADD filebeat-7.5.1-amd64.deb /tmp

RUN cd /tmp && yum localinstall -y filebeat-7.5.1-amd64.deb

# Set environment variables
ENV CATALINA_HOME /opt/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH


COPY ./index.html /$CATALINA_HOME/webapps/ROOT

# Expose the port that Tomcat runs on
EXPOSE 8080

# Set the working directory inside the container
WORKDIR $CATALINA_HOME

# Start Tomcat
CMD ["catalina.sh", "run"]

```

```bash
## 测试一下
docker run --rm -d -p 8088:8080 harbor.ceamg.com/baseimages/xinn-web1:x3
```



### 4.4 filebeat 配置

```bash
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/tomcat/logs/catalina.*.log 
  fields:
    type: tomcat-catalina
- type: log
  enabled: true
  paths:
    - /opt/tomcat/logs/localhost_access_log.*.txt
  fields: 
    type: tomcat-accesslog
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 1
setup.kibana:

output.kafka:
  hosts: ["192.168.149.26:9092"]
  required_acks: 1
  topic: "xinn-app1"
  compression: gzip
  max_message_bytes: 1000000
```







![image-20231208172004824](https://cdn1.ryanxin.live/image-20231208172004824.png)



![image-20231208172026399](https://cdn1.ryanxin.live/image-20231208172026399.png)

![image-20231205155610290](https://cdn1.ryanxin.live/image-20231205155610290.png)



![image-20231221101157108](https://cdn1.ryanxin.live/image-20231221101157108.png)





## 五、安装logstash 



```bash
apt install openjdk-11-jdk -y

root@logstash:/tmp# dpkg -i logstash-7.6.2.deb
Selecting previously unselected package logstash.
(Reading database ... 106223 files and directories currently installed.)
Preparing to unpack logstash-7.6.2.deb ...
Unpacking logstash (1:7.6.2-1) ...
Setting up logstash (1:7.6.2-1) ...
Using provided startup.options file: /etc/logstash/startup.options
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.headius.backport9.modules.Modules to method sun.nio.ch.NativeThread.signal(long)
WARNING: Please consider reporting this to the maintainers of com.headius.backport9.modules.Modules
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/pleaserun-0.0.30/lib/pleaserun/platform/base.rb:112: warning: constant ::Fixnum is deprecated
Successfully created system startup script for Logstash
```



### 5.1 测试从kafka中读并输出到标准输出

```bash
vim /etc/logstash/conf.d/kafkatoes.conf
input {
  kafka {
    bootstrap_servers => "192.168.149.26:9092,192.168.149.27:9092,192.168.149.28:9092"
    topics => ["xinn-app1"]
    codec => "json"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}

```



![image-20231221154105861](https://cdn1.ryanxin.live/image-20231221154105861.png)



### 5.2 从kafka中读并输出到ES集群

```bash
vim /etc/logstash/conf.d/kafkatoes.conf
input {
  kafka {
    bootstrap_servers => "192.168.149.26:9092,192.168.149.27:9092,192.168.149.28:9092"
    topics => ["xinn-app1"]
    codec => "json"
  }
}

output {
  if [fields][type] == "tomcat-accesslog" {
    elasticsearch {
    hosts => ["192.168.149.22:9200","192.168.149.23:9200","192.168.149.24:9200"]
    index => "xinn-app1-accesslog-%{+YYYY.MM.dd}"
    }
  }
  if [fields][type] == "tomcat-catalina" {
    elasticsearch {
    hosts => ["192.168.149.22:9200","192.168.149.23:9200","192.168.149.24:9200"]
    index => "xinn-app1-catalina-%{+YYYY.MM.dd}"
    }
  }


#  stdout {
#    codec => rubydebug
#  }
}
```



### 5.3 安装head插件查看数据

![image-20231221172155957](https://cdn1.ryanxin.live/image-20231221172155957.png)

![image-20231221172437806](https://cdn1.ryanxin.live/image-20231221172437806.png)

![image-20231221172519941](https://cdn1.ryanxin.live/image-20231221172519941.png)
