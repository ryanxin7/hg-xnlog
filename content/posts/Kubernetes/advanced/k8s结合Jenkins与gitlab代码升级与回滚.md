---
author: Ryan
title: k8s结合Jenkins与gitlab代码升级与回滚
date: 2023-12-04
lastmod: 2023-12-04
tags:
  - Kubernetes
  - 持续集成
categories:
  - Kubernetes
expirationReminder:
  enable: false
---

## k8s结合Jenkins与gitlab代码升级与回滚



![img](https://cdn1.ryanxin.live/0)



## 一、jenkins环境准备



### 1.1 安装jdk17

![image-20231129103240413](https://cdn1.ryanxin.live/image-20231129103240413.png)



```bash
tar -xf jdk-17.0.9_linux-x64.tar.gz -C /opt

## 配置环境变量
vim /etc/profile.d/java.sh
JAVA_HOME=/opt/jdk-17.0.9
CLASSPATH=.:$JAVA_HOME/lib/jrt-fs.jar
PATH=$PATH:$JAVA_HOME/bin

source /etc/profile.d/java.sh
```





```bash
##检查版本
root@ceamg-server:/opt/jdk-17.0.9# java -version
java version "17.0.9" 2023-10-17 LTS
Java(TM) SE Runtime Environment (build 17.0.9+11-LTS-201)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.9+11-LTS-201, mixed mode, sharing)
```





### 1.2 安装jenkins

下载：https://mirrors.tuna.tsinghua.edu.cn/jenkins/debian-stable/jenkins_2.414.3_all.deb

```bash
root@ceamg-server:/tmp# dpkg -i jenkins_2.414.3_all.deb
Selecting previously unselected package jenkins.
(Reading database ... 75661 files and directories currently installed.)
Preparing to unpack jenkins_2.414.3_all.deb ...
Unpacking jenkins (2.414.3) ...
dpkg: dependency problems prevent configuration of jenkins:
 jenkins depends on net-tools; however:
  Package net-tools is not installed.

dpkg: error processing package jenkins (--install):
 dependency problems - leaving unconfigured
Processing triggers for systemd (245.4-4ubuntu3.11) ...
Errors were encountered while processing:
 jenkins
 
## 出现依赖问题
root@ceamg-server:/tmp# dpkg -r jenkins
(Reading database ... 75675 files and directories currently installed.)
Removing jenkins (2.414.3) ...
```



```bash
## 安装依赖
root@ceamg-server:/tmp# apt install  net-tools
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  net-tools
0 upgraded, 1 newly installed, 0 to remove and 264 not upgraded.
Need to get 196 kB of archives.
After this operation, 864 kB of additional disk space will be used.
Get:1 http://mirrors.tuna.tsinghua.edu.cn/ubuntu focal/main amd64 net-tools amd64 1.60+git20180626.aebd88e-1ubuntu1 [196 kB]
Fetched 196 kB in 2s (121 kB/s)
Selecting previously unselected package net-tools.
(Reading database ... 75664 files and directories currently installed.)
Preparing to unpack .../net-tools_1.60+git20180626.aebd88e-1ubuntu1_amd64.deb ...
Unpacking net-tools (1.60+git20180626.aebd88e-1ubuntu1) ...
Setting up net-tools (1.60+git20180626.aebd88e-1ubuntu1) ...
Processing triggers for man-db (2.9.1-1) ...
```



```bash
## 在Jenkins中配置java路径

vim /lib/systemd/system/jenkins.service
# The Java home directory. When left empty, JENKINS_JAVA_CMD and PATH are consulted.
Environment="JAVA_HOME=/opt/jdk-17.0.9"
```



```bash
## 重启Jenkins
root@ceamg-server:/opt/jdk-17.0.9# systemctl restart jenkins.service
Warning: The unit file, source configuration file or drop-ins of jenkins.service changed on disk. Run 'systemctl daemon-reload' to reload units.
Job for jenkins.service failed because the control process exited with error code.
See "systemctl status jenkins.service" and "journalctl -xe" for details.
root@ceamg-server:/opt/jdk-17.0.9#
root@ceamg-server:/opt/jdk-17.0.9# systemctl daemon-reload
root@ceamg-server:/opt/jdk-17.0.9#
root@ceamg-server:/opt/jdk-17.0.9# systemctl restart jenkins.service
root@ceamg-server:/opt/jdk-17.0.9#
root@ceamg-server:/opt/jdk-17.0.9# systemctl status jenkins.service
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/jenkins.service.d
             └─override.conf
     Active: active (running) since Fri 2023-12-01 06:03:29 UTC; 49s ago
   Main PID: 1083179 (java)
      Tasks: 59 (limit: 9447)
     Memory: 2.1G
     CGroup: /system.slice/jenkins.service
             └─1083179 /opt/jdk-17.0.9/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=>

Dec 01 06:03:11 ceamg-server jenkins[1083179]: Jenkins initial setup is required. An admin user has been created and a password generated.
Dec 01 06:03:11 ceamg-server jenkins[1083179]: Please use the following password to proceed to installation:
Dec 01 06:03:11 ceamg-server jenkins[1083179]: aad0ca64ee9a499cb00ea803b0563694
Dec 01 06:03:11 ceamg-server jenkins[1083179]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Dec 01 06:03:11 ceamg-server jenkins[1083179]: *************************************************************
Dec 01 06:03:11 ceamg-server jenkins[1083179]: *************************************************************
Dec 01 06:03:11 ceamg-server jenkins[1083179]: *************************************************************
Dec 01 06:03:29 ceamg-server jenkins[1083179]: 2023-12-01 06:03:29.569+0000 [id=48]        INFO        jenkins.InitReactorRunner$1#onAttained: Complet>
Dec 01 06:03:29 ceamg-server jenkins[1083179]: 2023-12-01 06:03:29.584+0000 [id=26]        INFO        hudson.lifecycle.Lifecycle#onReady: Jenkins is >
Dec 01 06:03:29 ceamg-server systemd[1]: Started Jenkins Continuous Integration Server.


## 检查Jenkins服务启动情况
root@ceamg-server:/opt/jdk-17.0.9#
root@ceamg-server:/opt/jdk-17.0.9# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:41991           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:41643           0.0.0.0:*               LISTEN      625/rpc.mountd
tcp        0      0 0.0.0.0:54605           0.0.0.0:*               LISTEN      625/rpc.mountd
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/init
tcp        0      0 127.0.0.1:20180         0.0.0.0:*               LISTEN      82703/VMToolsAgentA
tcp        0      0 127.0.0.1:20181         0.0.0.0:*               LISTEN      82703/VMToolsAgentA
tcp        0      0 127.0.0.1:33397         0.0.0.0:*               LISTEN      82686/G2HProxy
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      621/systemd-resolve
tcp        0      0 127.0.0.1:20182         0.0.0.0:*               LISTEN      82686/G2HProxy
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1666/sshd: /usr/sbi
tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      29129/sshd: ceamg@p
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:58913           0.0.0.0:*               LISTEN      625/rpc.mountd
tcp6       0      0 :::34469                :::*                    LISTEN      625/rpc.mountd
tcp6       0      0 :::35597                :::*                    LISTEN      -
tcp6       0      0 :::60909                :::*                    LISTEN      625/rpc.mountd
tcp6       0      0 :::111                  :::*                    LISTEN      1/init
tcp6       0      0 :::8080                 :::*                    LISTEN      1083179/java

```

![image-20231201143838772](https://cdn1.ryanxin.live/image-20231201143838772.png)



```bash
## 获取管理员密码
root@ceamg-server:/opt/jdk-17.0.9# cat /var/lib/jenkins/secrets/initialAdminPassword
aad0ca64ee9a499cb00ea803b0563694
```



### 1.3 初始化Jenkins

![image-20231201144616862](https://cdn1.ryanxin.live/image-20231201144616862.png)



![image-20231201144709325](https://cdn1.ryanxin.live/image-20231201144709325.png)



### 1.4 创建任务测试Jenkins

![image-20231201154407514](https://cdn1.ryanxin.live/image-20231201154407514.png)

![image-20231201154614638](https://cdn1.ryanxin.live/image-20231201154614638.png)



### 1.5 安装Jenkins gitlab插件

![image-20231201155058023](https://cdn1.ryanxin.live/image-20231201155058023.png)

![image-20231201155159325](https://cdn1.ryanxin.live/image-20231201155159325.png)



### 1.6 创建基于参数化构建

![image-20231201161908459](https://cdn1.ryanxin.live/image-20231201161908459.png)



![image-20231201162914825](https://cdn1.ryanxin.live/image-20231201162914825.png)

![](https://cdn1.ryanxin.live/image-20231201171418209.png)



### 1.7 创建Jenkins与Gitlab基于密钥免密

```bash
root@ceamg-server:/data# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:aZCal90HsZPHoIcJj4ggYFdSDU3UJ51+shAMTOt4bVU root@ceamg-server
The key's randomart image is:
+---[RSA 3072]----+
|=. oo+X=+ + .E   |
|o..... O.O X.    |
|  . . +.= @.o    |
|     oo+.=.* .   |
|    o.ooSoo =    |
|     ....  o     |
|                 |
|                 |
|                 |
+----[SHA256]-----+
root@ceamg-server:/data#
root@ceamg-server:/data# cat /root/.ssh/
authorized_keys  id_rsa           id_rsa.pub
root@ceamg-server:/data# cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDNRtj7OYSloUzmkhC/o9IohtoqKT9hhXlfM9t8VxnQZKqoCSlnZrlrpSk32Ds9JcKfpytu5x2iwPsF/Ytb8wS4dTBKc/89FyO+U1cOMxXakNBL9lYIbyxNrVs2GAWRwJyyYsfa8kOTLvkPejRm2d5HeQKLqbbTChHsubFQDHDLjz0DVNPx9J5t4Z33Dn8cSjAjMtKVLCgb810gOW0S60RMFLSSmoEzDwWG4Kv7vbOgugNDhQS3UVfOY1DU6/b8OMrWzaPLySaUXe0CjDf3sDgMU3LG2CsHaJ9eIqBwI5YQ84RanCINfbuyrlFW4Mu3FF1B2lxfZlen/O0vcfOq0sJX990/1xO136xkngWzElOEIkCpVCUqMyIj4BS0qdYu+IUw3+diKcsOC9nbJ1E3zootRiTUD6oejbAB1aSy4g0JaavfGJjpoJXLSWV09wk8Osivrf3MXme7vhzLNzF9vraiwenNUM/BAvYSVAJQyQNvksGFrLFm4U9KXG5WqpDLTP8= root@ceamg-server
```



![image-20231201163530046](https://cdn1.ryanxin.live/image-20231201163530046.png)



### 1.8 测试代码Clone

```bash
root@ceamg-server:/opt# git clone  http://10.1.0.140/xxteam/xinn.git
Cloning into 'xinn'...
Username for 'http://10.1.0.140': xin1
Password for 'http://xin1@10.1.0.140':
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), 3.03 KiB | 57.00 KiB/s, done.
```



```bash
root@ceamg-server:/opt# rm -rf xinn/
root@ceamg-server:/opt#
root@ceamg-server:/opt# git clone git@10.1.0.140:xxteam/xinn.git
Cloning into 'xinn'...
The authenticity of host '10.1.0.140 (10.1.0.140)' can't be established.
ECDSA key fingerprint is SHA256:lhRjKQBhgEhjbqcfKBb6oyle8C9EIOzu48QUoaeISIE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.1.0.140' (ECDSA) to the list of known hosts.
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
root@ceamg-server:/opt#
root@ceamg-server:/opt# rm -rf xinn/
root@ceamg-server:/opt#
root@ceamg-server:/opt# git clone git@10.1.0.140:xxteam/xinn.git
Cloning into 'xinn'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
```







## 二、Gitlab环境准备



### 2.1 安装Gitlib CE

下载:https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/focal/main/g/gitlab-ce/gitlab-ce_16.5.2-ce.0_amd64.deb

```bash
root@ceamg-server:~# dpkg -i /tmp/gitlab-ce_16.5.2-ce.0_amd64.deb
Selecting previously unselected package gitlab-ce.
(Reading database ... 75661 files and directories currently installed.)
Preparing to unpack .../gitlab-ce_16.5.2-ce.0_amd64.deb ...
Unpacking gitlab-ce (16.5.2-ce.0) ...
Setting up gitlab-ce (16.5.2-ce.0) ...
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.



     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/


Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

Help us improve the installation experience, let us know how we did with a 1 minute survey:
https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=omnibus&release=16-5
```



### 2.2 修改Gitlab 对外发布地址

```bash
vim /etc/gitlab/gitlab.rb
external_url 'http://10.1.0.140'
```

```bash
##更新配置
gitlab-ctl reconfigure
```





```bash
## 获取root密码
root@gitlab:~# cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: I5qRTkn+LrdefGIo43ifT0cBHQUuEAHoJDuZuaS4CG8=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```



![image-20231201155418428](https://cdn1.ryanxin.live/image-20231201155418428.png)



### 2.3 创建项目和组



![image-20231201155619418](https://cdn1.ryanxin.live/image-20231201155619418.png)

![image-20231201155704811](https://cdn1.ryanxin.live/image-20231201155704811.png)

![image-20231201155912403](https://cdn1.ryanxin.live/image-20231201155912403.png)

![image-20231201160625876](https://cdn1.ryanxin.live/image-20231201160625876.png)

![image-20231201160735931](https://cdn1.ryanxin.live/image-20231201160735931.png)

![image-20231201163209885](https://cdn1.ryanxin.live/image-20231201163209885.png)

<br>



## 三、持续继承与部署



### 3.1  构建JDK17 镜像

#### 3.1.1 下载JDK17程序包

![](https://cdn1.ryanxin.live/image-20231129103240413.png)

#### 3.1.2 准备Dockerfile文件

```dockerfile
FROM ubuntu:latest
MAINTAINER xinn.cc
WORKDIR /usr/local/java17
ADD ./jdk-17.0.9_linux-x64.tar.gz  /usr/local/java17/
ENV JAVA_HOME=/usr/local/java17/jdk-17.0.9
ENV CLASSPATH=.:$JAVA_HOME/lib/jrt-fs.jar
ENV PATH=$PATH:$JAVA_HOME/bin
```



#### 3.1.3 准备构建脚本

```bash
#!/bin/bash
TAG=$1
docker  build -t harbor.ceamg.com/baseimages/jdk17_0.9:${TAG} .
docker push harbor.ceamg.com/baseimages/jdk17_0.9:${TAG}
```





#### 3.1.4 制作镜像

```bash
root@harbor01[11:10:31]/dockerfile/jdk-17.0.9 #:ls
build_image_command.sh  Dockerfile  jdk-17.0.9_linux-x64.tar.gz
root@harbor01[11:10:33]/dockerfile/jdk-17.0.9 #:
root@harbor01[11:10:33]/dockerfile/jdk-17.0.9 #:sh build_image_command.sh x1
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  182.5MB
Step 1/7 : FROM ubuntu:latest
 ---> 6b7dfa7e8fdb
Step 2/7 : MAINTAINER xinn.cc
 ---> Running in 3aed1c94bf06
Removing intermediate container 3aed1c94bf06
 ---> 4d15526cf2a0
Step 3/7 : WORKDIR /usr/local/java17
 ---> Running in a86d42bd6a1a
Removing intermediate container a86d42bd6a1a
 ---> d7c03d0d10cd
Step 4/7 : ADD ./jdk-17.0.9_linux-x64.tar.gz  /usr/local/java17/
 ---> 881b04611e31
Step 5/7 : ENV JAVA_HOME=/usr/local/java/jdk-17.0.9
 ---> Running in 6703b928fb0d
Removing intermediate container 6703b928fb0d
 ---> dbb88d39fcb0
Step 6/7 : ENV CLASSPATH=.:$JAVA_HOME/lib/jrt-fs.jar
 ---> Running in 2ee6e6746a43
Removing intermediate container 2ee6e6746a43
 ---> a89b458baee1
Step 7/7 : ENV PATH=$PATH:$JAVA_HOME/bin
 ---> Running in 6feaa6d5e20c
Removing intermediate container 6feaa6d5e20c
 ---> 13b6bed2c720
Successfully built 13b6bed2c720
Successfully tagged harbor.ceamg.com/baseimages/jdk17_0.9:x1
The push refers to repository [harbor.ceamg.com/baseimages/jdk17_0.9]
c807951d4c31: Pushed
b524ae647412: Pushed
6515074984c6: Mounted from baseimages/ubuntu
x1: digest: sha256:e43e2bf6ff777ea917f274f26abcc060a369be765ecba6be46e4aa89ff7a17ad size: 949
```





#### 3.1.4 测试镜像

```bash
##临时运行容器
docker run -it harbor.ceamg.com/baseimages/jdk17_0.9:x1

##进入容器查看java版本
root@17b96c96a4aa:/usr/local/java17# java -version
java version "17.0.9" 2023-10-17 LTS
Java(TM) SE Runtime Environment (build 17.0.9+11-LTS-201)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.9+11-LTS-201, mixed mode, sharing)
```



### 3.2 构建tomcat 基础业务镜像

#### 3.2.1 下载Tomcat程序包

![image-20231204165359159](https://cdn1.ryanxin.live/image-20231204165359159.png)

#### 3.2.2 准备Dockerfile文件

```docker
# Use a base image with Java 17 and Tomcat
FROM harbor.ceamg.com/baseimages/jdk17_0.9:x1

# Tomcat version to be installed
ENV TOMCAT_VERSION 10.1.16

COPY ./apache-tomcat-10.1.16.tar.gz /tmp/

# 在容器中解压缩Tomcat文件
RUN tar -xf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz -C /opt/ && \
    mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \
    rm apache-tomcat-${TOMCAT_VERSION}.tar.gz


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

#### 3.2.3 制作镜像

```bash
root@harbor01[14:40:49]/dockerfile/tomcat17 #:sh build_image_command.sh x1
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  12.49MB
Step 1/10 : FROM harbor.ceamg.com/baseimages/jdk17_0.9:x1
 ---> 2831b54b6401
Step 2/10 : ENV TOMCAT_VERSION 10.1.16
 ---> Using cache
 ---> b08960efe9a8
Step 3/10 : COPY ./apache-tomcat-10.1.16.tar.gz /tmp/
 ---> Using cache
 ---> 7d3ddba62f34
Step 4/10 : RUN tar -xf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz -C /opt/ &&     mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat &&     rm -rf /tmp/apache-tomcat-${TOMCAT_VERSION}.tar.gz
 ---> Running in ab9db765a511
Removing intermediate container ab9db765a511
 ---> be00e79b1f2d
Step 5/10 : ENV CATALINA_HOME /opt/tomcat
 ---> Running in 8428bb9c55e7
Removing intermediate container 8428bb9c55e7
 ---> 75ee07ece0e3
Step 6/10 : ENV PATH $CATALINA_HOME/bin:$PATH
 ---> Running in fbe785a46542
Removing intermediate container fbe785a46542
 ---> c9b94571fb95
Step 7/10 : COPY ./index.html /$CATALINA_HOME/webapps/
 ---> 4c1b4b949844
Step 8/10 : EXPOSE 8080
 ---> Running in c66aafe5856d
Removing intermediate container c66aafe5856d
 ---> 466e08e303eb
Step 9/10 : WORKDIR $CATALINA_HOME
 ---> Running in 2a6b3b867c19
Removing intermediate container 2a6b3b867c19
 ---> 79cf969913dc
Step 10/10 : CMD ["catalina.sh", "run"]
 ---> Running in d465b3338a80
Removing intermediate container d465b3338a80
 ---> 3b7659ab3a5c
Successfully built 3b7659ab3a5c
Successfully tagged harbor.ceamg.com/baseimages/webapp-tomcat:x1
The push refers to repository [harbor.ceamg.com/baseimages/webapp-tomcat]
83f5ae443ab3: Pushed
ead28053bfb3: Pushed
a256fbd2326d: Pushed
c807951d4c31: Mounted from baseimages/jdk17_0.9
b524ae647412: Mounted from baseimages/jdk17_0.9
6515074984c6: Mounted from baseimages/jdk17_0.9
x1: digest: sha256:c0fae36668191a3950f37e6a54dbacb090b5ccddaef16e8c74083df9c534c4ac size: 1580
```



#### 3.2.4 测试业务镜像

```bash 
## 测试运行tomcat镜像
docker run --rm -d -p 8080:8080 harbor.ceamg.com/baseimages/webapp-tomcat:x1

##停止并删除这个临时运行的容器
docker stop 593673c9d

##进入容器
docker exec -it 593673c9d bash
```



### 3.3  准备业务自动化构建文件

#### 3.3.1 创建服务文件目录

```bash
mkdir /opt/k8s-data/yaml/ -r
mkdir /opt/k8s-data/dockerfile/xinn -r
```



#### 3.3.2 创建Tomcat 业务deployment文件

```yaml
vim /opt/k8s-data/yaml/xinn-web1.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: xinn-web1-deployment
  namespace: xinn
spec:
  replicas: 3
  selector:
    matchLabels:
      app: xinn-web1-tomcat
  template:
    metadata:
      labels:
        app: xinn-web1-tomcat
    spec:
      containers:
      - name: xinn-web1
        image: harbor.ceamg.com/baseimages/webapp-tomcat:x1
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: xinn-web1-service
  namespace: xinn
spec:
  selector:
    app: xinn-web1-tomcat
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 9988
  type: NodePort
```



#### 3.3.3 创建Jenkins自动化构建脚本

```bash
#!/bin/bash
#Author: ZhangShiJie
#Date: 2019-08-03
#Version: v1
#记录脚本开始执行时间
starttime=$(date +'%Y-%m-%d %H:%M:%S')
#变量
SHELL_DIR="/root/scripts"
SHELL_NAME="$0"
K8S_CONTROLLER1="10.1.0.32"
K8S_CONTROLLER2="10.1.0.33"
DATE=$(date +%Y-%m-%d_%H_%M_%S)
METHOD=$1
Branch=$2
if test -z $Branch; then
    Branch=develop
fi
function Code_Clone() {
    Git_URL="git@10.1.0.140:xxteam/xinn.git"
    DIR_NAME=$(echo ${Git_URL} | awk -F "/" '{print $2}' | awk -F "." '{print $1}')
    DATA_DIR="/data/gitdata/"
    Git_Dir="${DATA_DIR}/${DIR_NAME}"
    cd ${DATA_DIR} && echo "即将清空上一版本代码并获取当前分支最新代码" && sleep 1 && rm -rf ${DIR_NAME}
    echo "即将开始从分支${Branch} 获取代码" && sleep 1
    git clone -b ${Branch} ${Git_URL}
    echo "分支${Branch} 克隆完成，即将进行代码编译!" && sleep 1
    #cd ${Git_Dir} && mvn clean package
    #echo "代码编译完成，即将开始将IP地址等信息替换为测试环境"
    #####################################################
    sleep 1
    cd ${Git_Dir}
    tar czf ${DIR_NAME}.tar.gz ./*
}

#将打包好的压缩文件拷贝到k8s 控制端服务器
function Copy_File() {
    echo "压缩文件打包完成，即将拷贝到k8s 控制端服务器${K8S_CONTROLLER1}" && sleep 1
    scp ${Git_Dir}/${DIR_NAME}.tar.gz root@${K8S_CONTROLLER1}:/opt/k8s-data/dockerfile/xinn
    ssh root@${K8S_CONTROLLER1} "cd /opt/k8s-data/dockerfile/xinn && tar xf ${DIR_NAME}.tar.gz ./"
    echo "压缩文件拷贝完成,服务器${K8S_CONTROLLER1}即将开始制作Docker 镜像!" && sleep 1
}

#到控制端执行脚本制作并上传镜像
function Make_Image() {
    echo "开始制作Docker镜像并上传到Harbor服务器" && sleep 1
    ssh root@${K8S_CONTROLLER1} "cd /opt/k8s-data/dockerfile/xinn && bash build-command.sh ${DATE}"
    echo "Docker镜像制作完成并已经上传到harbor服务器" && sleep 1
}

#到控制端更新k8s yaml文件中的镜像版本号,从而保持yaml文件中的镜像版本号和k8s中版本号一致
function Update_k8s_yaml() {
    echo "即将更新k8s yaml文件中镜像版本" && sleep 1
    ssh root@${K8S_CONTROLLER1} "cd /opt/k8s-data/yaml && sed -i 's/image: harbor.ceamg.*/image: harbor.ceamg.com\/baseimages\/xinn-web1:${DATE}/g' xinn-web1.yaml"
    echo "k8s yaml文件镜像版本更新完成,即将开始更新容器中镜像版本" && sleep 1
}

#到控制端更新k8s中容器的版本号,有两种更新办法，一是指定镜像版本更新，二是apply执行修改过的yaml文件
function Update_k8s_container() {
    #第一种方法
    #ssh root@${K8S_CONTROLLER1} "kubectl set image deployment/linux36-nginx-deployment linux36-nginx-container=harbor.magedu.net/linux36/nginx-web1:${DATE} -n linux36"
    #第二种方法,推荐使用第一种
    ssh root@${K8S_CONTROLLER1} "cd /opt/k8s-data/yaml && kubectl apply -f xinn-web1.yaml --record"
    echo "k8s 镜像更新完成" && sleep 1
    echo "当前业务镜像版本: harbor.ceamg.com/baseimages/xinn-web1:${DATE}"
    #计算脚本累计执行时间，如果不需要的话可以去掉下面四行
    endtime=$(date +'%Y-%m-%d %H:%M:%S')
    start_seconds=$(date --date="$starttime" +%s)
    end_seconds=$(date --date="$endtime" +%s)
    echo "本次业务镜像更新总计耗时："$((end_seconds - start_seconds))"s"
}

#基于k8s 内置版本管理回滚到上一个版本
function rollback_last_version() {
    echo "即将回滚之上一个版本"
    ssh root@${K8S_CONTROLLER1} "kubectl rollout undo deployment/xinn-web1-deployment -n xinn"
    sleep 1
    echo "已执行回滚至上一个版本"
}

#使用帮助
usage() {
    echo "部署使用方法为 ${SHELL_DIR}/${SHELL_NAME} deploy "
    echo "回滚到上一版本使用方法为 ${SHELL_DIR}/${SHELL_NAME} rollback_last_version"
}

#主函数
main() {
    case ${METHOD} in
    deploy)
        Code_Clone
        Copy_File
        Make_Image
        Update_k8s_yaml
        Update_k8s_container
        ;;
    rollback_last_version)
        rollback_last_version
        ;;
    *)
        usage
        ;;
    esac
}

main $1 $2 $3
```



#### 3.3.4 测试K8S节点访问harbor拉取镜像

```bash
root@k8s-made-01-32:/etc/docker/certs.d# tree ../certs.d/
../certs.d/
└── harbor.ceamg.com
    ├── ca.crt
    ├── harbor.ceamg.com.cert
    ├── harbor.ceamg.com.crt
    ├── harbor.ceamg.com.key
    └── hosts.toml
```



```bash
root@k8s-made-01-32:/opt/k8s-data/dockerfile/xinn# docker login harbor.ceamg.com
Username: xcd
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```



## 3.4 创建任务流水线实现自动化部署

#### 3.4.1 配置流水线参数

![image-20231204170355263](https://cdn1.ryanxin.live/image-20231204170355263.png)

![image-20231204170413225](https://cdn1.ryanxin.live/image-20231204170413225.png)

#### 3.4.2 更改Gitlab中项目代码

![image-20231204170623240](https://cdn1.ryanxin.live/image-20231204170623240.png)



#### 3.4.3 执行项目构建

![image-20231204170712275](https://cdn1.ryanxin.live/image-20231204170712275.png)



#### 3.5.4 项目构建过程

![image-20231204170751079](https://cdn1.ryanxin.live/image-20231204170751079.png)

![image-20231204170832969](https://cdn1.ryanxin.live/image-20231204170832969.png)



#### 3.5.5 服务访问测试

![image-20231204170909547](https://cdn1.ryanxin.live/image-20231204170909547.png)