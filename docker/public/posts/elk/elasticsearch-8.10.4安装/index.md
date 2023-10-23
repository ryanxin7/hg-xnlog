# Elasticsearch8.10.4搭建部署

## 1.环境准备

### 1.1 集群规划

| 服务器                | 角色            |
| --------------------- | --------------- |
| 192.168.10.107,eslg01 | master/data节点 |
| 192.168.10.108,eslg02 | master/data节点 |
| 192.168.10.109,eslg03 | master/data节点 |



>es各节点均为master，Elasticsearch-8版本部署的集群无主模式,这里使用了三台全新的机器，考虑到es版本8对java的要求相对较高，如机器部署的应用较多，避免java环境混乱以及应用之间相互影响，所以es8不建议使用在已经部署较多java环境的应用机器。





**ES各版本对java版本的需求：**

    ES 7.x 及之前版本：选择 Java 8
    
    ES 8.x，支持 Java 17 和 Java 18，推荐版本：
        其中对于ES 8.0：Java版本仅支持 Java 17
        ES 8.1及以上版本：支持Java 17 以及 Java 18，建议使用Java 17,因为对应版本的 Logstash 不支持 Java 18

【注意】

```bash
Java 9、Java 10、Java 12 和 Java 13 均为官方公布的短期版本，ES各版本均不推荐使用这几个
elasticsearch项目的jdk目录下现在已经内置了openjdk18，也可以直接使用
```



由于es和jdk是一个强依赖的关系，所以当我们在新版本的ElasticSearch压缩包中包含有自带的jdk，但是当我们的Linux中已经安装了jdk之后，就会发现启动es的时候优先去找的是Linux中已经装好的jdk，此时如果jdk的版本不一致，就会造成jdk不能正常运行，报错如下：

```bash
warning: usage of JAVA_HOME is deprecated, use ES_JAVA_HOME
Future versions of Elasticsearch will require Java 11; your Java version from [/usr/local/jdk1.8.0_291/jre] does not meet this requirement. Consider switching to a distribution of Elasticsearch with a bundled JDK. If you are already using a distribution with a bundled JDK, ensure the JAVA_HOME environment variable is not set.

```



注：如果Linux服务本来没有配置jdk，则会直接使用es目录下默认的jdk，反而不会报错



**添加配置解决jdk版本问题**

```bash
#进入bin目录
cd /usr/local/elasticsearch-7.15.2/bin
vim ./elasticsearch 
```



```bash
# 将jdk修改为es中自带jdk的配置目录
export JAVA_HOME=/usr/local/softwore/elasticsearch-7.15.2/jdk
export PATH=$JAVA_HOME/bin:$PATH

if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/usr/local/softwore/elasticsearch-7.15.2/jdk/bin/java"
else
        JAVA=`which java`
fi
```



### 1.2 各节点设置主机名

```bash
#192.168.10.107 
hostnamectl set-hostname eslg01

#192.168.10.107 
hostnamectl set-hostname eslg01

#192.168.10.107 
hostnamectl set-hostname eslg01
```



### 1.3 各节点创建普通用户

>ES不能使用root用户来启动，否则会报错，使用普通用户来安装启动。创建一个普通用户以及定义一些常规目录用于存放我们的数据文件以及安装包等
```bash
# 3台机器都需要执行
useradd essl && echo "essl:Ceamg.com" | chpasswd
```



### 1.4 各节点将普通用户权限提高

```bash
[root@es01 ~]# visudo
# 增加一行普通用户权限内容
essl ALL=(ALL) NOPASSWD:ALL
```

>NOPASSWD 允许用户在执行被授权的命令时，不需要输入密码验证。这通常用于简化特定任务的自动化，但需要特别小心，确保仅分配给受信任的用户。




### 1.5 各节点添加hosts主机名解析
```bash
vim /etc/hosts
192.168.10.107 eslg01
192.168.10.108 eslg02
192.168.10.109 eslg03
```
如果在各节点的/etc/hosts中都配置了节点的ip解析，那后续在配置文件中，相关的ip配置都可以用解析名代替






### 1.6 安装包准备

Releases: https://www.elastic.co/cn/downloads/past-releases#elasticsearch

<img src="https://cdn1.ryanxin.live/image-20231020133656380.png" alt="image-20231020133656380" style="zoom:80%;" />





### 1.7 各节点环境优化(各节点)
注意环境优化为各节点均要执行的实施操作，需要自行在每个节点去执行命令

#### 1.7.1 优化1：最大文件数
系统允许 Elasticsearch 打开的最大文件数需要修改成65536
```bash
[root@es01 ~]# vim /etc/security/limits.conf
# End of file
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 65536

# 断开重连会话
[root@es01 ~]# ulimit -n
65536

```

>如果这个配置不优化后续启动服务会出现：
[error] max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536] elasticsearch



#### 1.7.2 优化2：最大进程数

允许最大进程数配置修该成4096；不是4096则需要修改优化

```bash
[root@es01 ~]# vim /etc/security/limits.d/20-nproc.conf
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     4096
root       soft    nproc     unlimited
```



>这个配置不优化启动服务会出现：
> [error]max number of threads [1024] for user [judy2] likely too low, increase to at least [4096]



#### 1.7.3 优化3：虚拟内存

```bash
# 增加配置项vm.max_map_count
[root@es01 ~]# vim /etc/sysctl.conf 
vm.max_map_count=262144
# 重载配置
[root@es01 ~]# sysctl -p
```



>这个配置不优化启动服务会出现：
> [error]max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]





## 2.安装Elasticsearch

### 2.1 解压安装

```sh
[root@es01 ~]# mkdir -p /opt/module/
[root@es01 ~]# chown -R wangting.wangting /opt/module/
[root@es01 ~]# tar -xf elasticsearch-8.3.2-linux-x86_64.tar.gz -C /opt/module/
[root@es01 ~]# chown -R wangting.wangting /opt/module/elasticsearch-8.3.2
[root@es01 ~]# cd /opt/module/elasticsearch-8.3.2/
[root@es01 elasticsearch-8.3.2]# ls -l
total 892
drwxr-xr-x  2 wangting wangting   4096 Jul  6  2022 bin
drwxr-xr-x  3 wangting wangting   4096 Feb 10 15:05 config
drwxr-xr-x  8 wangting wangting   4096 Jul  6  2022 jdk
drwxr-xr-x  5 wangting wangting   4096 Jul  6  2022 lib
-rw-r--r--  1 wangting wangting   3860 Jul  6  2022 LICENSE.txt
drwxr-xr-x  2 wangting wangting   4096 Jul  6  2022 logs
drwxr-xr-x 66 wangting wangting   4096 Jul  6  2022 modules
-rw-r--r--  1 wangting wangting 874704 Jul  6  2022 NOTICE.txt
drwxr-xr-x  2 wangting wangting   4096 Jul  6  2022 plugins
-rw-r--r--  1 wangting wangting   2710 Jul  6  2022 README.asciidoc
```



### 2.2 配置环境变量

```sh
[root@es01 elasticsearch-8.3.2]# vim /etc/profile
# ES
export JAVA_HOME=/opt/module/elasticsearch-8.3.2/jdk
export ES_HOME=/opt/module/elasticsearch-8.3.2
export PATH=$PATH:$ES_HOME/bin

# 引用
[root@es01 elasticsearch-8.3.2]# source /etc/profile
[root@es01 ~]# /opt/module/elasticsearch-8.3.2/jdk/bin/java -version
openjdk version "18.0.1.1" 2022-04-22
OpenJDK Runtime Environment (build 18.0.1.1+2-6)
OpenJDK 64-Bit Server VM (build 18.0.1.1+2-6, mixed mode, sharing)

# ###节点es02上操作###
[root@es02 ~]# vim /etc/profile
# ES
export JAVA_HOME=/opt/module/elasticsearch-8.3.2/jdk
export ES_HOME=/opt/module/elasticsearch-8.3.2
export PATH=$PATH:$ES_HOME/bin

[root@es02 ~]# source /etc/profile

# ###节点es03上操作###
[root@es03 ~]# vim /etc/profile
# ES
export JAVA_HOME=/opt/module/elasticsearch-8.3.2/jdk
export ES_HOME=/opt/module/elasticsearch-8.3.2
export PATH=$PATH:$ES_HOME/bin

[root@es03 ~]# source /etc/profile

```



### 2.3 创建es相关目录

```bash
# 创建数据文件目录
[root@es01 ~]# mkdir -p /opt/module/elasticsearch-8.3.2/data
# 创建证书生成目录
[root@es01 ~]# mkdir -p /opt/module/elasticsearch-8.3.2/config/certs
# 目录有改动，重新刷一下权限
[root@es01 module]# chown -R wangting:wangting /opt/module/elasticsearch-8.3.2

# 分发至es02、es03
# 新建目录
[root@es02 ~]# mkdir -p /opt/module
[root@es03 ~]# mkdir -p /opt/module
# 分发到其他节点,并在02 03上查看操作文件权限
[root@es01 ~]# scp -r /opt/module/elasticsearch-8.3.2 es02:/opt/module/elasticsearch-8.3.2
[root@es01 ~]# scp -r /opt/module/elasticsearch-8.3.2 es03:/opt/module/elasticsearch-8.3.2
# 目录文件权限刷用户所属
[root@es02 module]# chown -R wangting:wangting /opt/module/elasticsearch-8.3.2
[root@es03 module]# chown -R wangting:wangting /opt/module/elasticsearch-8.3.2
```



### 2.4 证书签发

```bash
# 在第一台服务器节点es01 设置集群多节点通信密钥
# 切换普通用户实施
[root@es01 module]# su - wangting
[wangting@es01 ~]$ cd /opt/module/elasticsearch-8.3.2/bin
[wangting@es01 bin]$ ./elasticsearch-certutil ca
warning: ignoring JAVA_HOME=/opt/module/elasticsearch-8.3.2/jdk; using bundled JDK
This tool assists you in the generation of X.509 certificates and certificate
signing requests for use with SSL/TLS in the Elastic stack.
The 'ca' mode generates a new 'certificate authority'
This will create a new X.509 certificate and private key that can be used
to sign certificate when running in 'cert' mode.

Use the 'ca-dn' option if you wish to configure the 'distinguished name'
of the certificate authority

By default the 'ca' mode produces a single PKCS#12 output file which holds:
    * The CA certificate
    * The CAs private key

If you elect to generate PEM format certificates (the -pem option), then the output will
be a zip file containing individual files for the CA certificate and private key

Please enter the desired output file [elastic-stack-ca.p12]: # 回车即可
Enter password for elastic-stack-ca.p12 :  # 回车即可

# 用 ca 证书签发节点证书，过程中需按三次回车键,生成目录：es的home:/opt/elasticsearch-8.3.2/
[wangting@es01 bin]$ ./elasticsearch-certutil cert --ca elastic-stack-ca.p12

If you specify any of the following options:
    * -pem (PEM formatted output)
    * -multiple (generate multiple certificates)
    * -in (generate certificates from an input file)
then the output will be be a zip file containing individual certificate/key files

Enter password for CA (elastic-stack-ca.p12) :  # 回车即可
Please enter the desired output file [elastic-certificates.p12]:  # 回车即可
Enter password for elastic-certificates.p12 :  # 回车即可

Certificates written to /opt/module/elasticsearch-8.3.2/elastic-certificates.p12

This file should be properly secured as it contains the private key for 
your instance.
This file is a self contained file and can be copied and used 'as is'
For each Elastic product that you wish to configure, you should copy
this '.p12' file to the relevant configuration directory
and then follow the SSL configuration instructions in the product guide.

For client applications, you may only need to copy the CA certificate and
configure the client to trust this certificate.

# 将生成的证书文件移动到 config/certs 目录中
[wangting@es01 bin]$ cd /opt/module/elasticsearch-8.3.2/
[wangting@es01 elasticsearch-8.3.2]$ ls -l | grep "elastic-"
-rw-------  1 wangting wangting   3596 Feb 10 16:05 elastic-certificates.p12
-rw-------  1 wangting wangting   2672 Feb 10 16:03 elastic-stack-ca.p12
[wangting@es01 elasticsearch-8.3.2]$ 
[wangting@es01 elasticsearch-8.3.2]$ mv elastic-certificates.p12 config/certs/
[wangting@es01 elasticsearch-8.3.2]$ mv elastic-stack-ca.p12 config/certs/
```



### 2.5 设置集群多节点 HTTP 证书

```bash
# 签发 Https 证书
[wangting@es01 elasticsearch-8.3.2]$ cd /opt/module/elasticsearch-8.3.2/bin/
[wangting@es01 bin]$ ./elasticsearch-certutil http
warning: ignoring JAVA_HOME=/opt/module/elasticsearch-8.3.2/jdk; using bundled JDK

## Elasticsearch HTTP Certificate Utility
The 'http' command guides you through the process of generating certificates
for use on the HTTP (Rest) interface for Elasticsearch.
This tool will ask you a number of questions in order to generate the right
set of files for your needs.
## Do you wish to generate a Certificate Signing Request (CSR)?
A CSR is used when you want your certificate to be created by an existing
Certificate Authority (CA) that you do not control (that is, you do not have
access to the keys for that CA). 
If you are in a corporate environment with a central security team, then you
may have an existing Corporate CA that can generate your certificate for you.
Infrastructure within your organisation may already be configured to trust this
CA, so it may be easier for clients to connect to Elasticsearch if you use a
CSR and send that request to the team that controls your CA.
If you choose not to generate a CSR, this tool will generate a new certificate
for you. That certificate will be signed by a CA under your control. This is a
quick and easy way to secure your cluster with TLS, but you will need to
configure all your clients to trust that custom CA.
######################################################
# 是否生成CSR，选择 N ，不需要                           #
######################################################
Generate a CSR? [y/N]N

## Do you have an existing Certificate Authority (CA) key-pair that you wish to use to sign your certificate?

If you have an existing CA certificate and key, then you can use that CA to
sign your new http certificate. This allows you to use the same CA across
multiple Elasticsearch clusters which can make it easier to configure clients,
and may be easier for you to manage.

If you do not have an existing CA, one will be generated for you.
######################################################
# 是否使用已经存在的CA证书，选择 y ，因为已经创建签发好了CA    #
######################################################
Use an existing CA? [y/N]y

## What is the path to your CA?
Please enter the full pathname to the Certificate Authority that you wish to
use for signing your new http certificate. This can be in PKCS#12 (.p12), JKS
(.jks) or PEM (.crt, .key, .pem) format.
######################################################
# 指定CA证书的路径地址，CA Path:后写绝对路径               #
######################################################
CA Path: /opt/module/elasticsearch-8.3.2/config/certs/elastic-stack-ca.p12
Reading a PKCS12 keystore requires a password.
It is possible for the keystore's password to be blank,
in which case you can simply press <ENTER> at the prompt

######################################################
# 设置密钥库的密码，直接 回车 即可                         #
######################################################
Password for elastic-stack-ca.p12:

## How long should your certificates be valid?

Every certificate has an expiry date. When the expiry date is reached clients
will stop trusting your certificate and TLS connections will fail.
Best practice suggests that you should either:
(a) set this to a short duration (90 - 120 days) and have automatic processes
to generate a new certificate before the old one expires, or
(b) set it to a longer duration (3 - 5 years) and then perform a manual update
a few months before it expires.

You may enter the validity period in years (e.g. 3Y), months (e.g. 18M), or days (e.g. 90D)
######################################################
# 设置证书的失效时间，这里的y表示年，5y则代表失效时间5年       #
######################################################
For how long should your certificate be valid? [5y] 5y

## Do you wish to generate one certificate per node?

If you have multiple nodes in your cluster, then you may choose to generate a
separate certificate for each of these nodes. Each certificate will have its
own private key, and will be issued for a specific hostname or IP address.

Alternatively, you may wish to generate a single certificate that is valid
across all the hostnames or addresses in your cluster.

If all of your nodes will be accessed through a single domain
(e.g. node01.es.example.com, node02.es.example.com, etc) then you may find it
simpler to generate one certificate with a wildcard hostname (*.es.example.com)
and use that across all of your nodes.

However, if you do not have a common domain name, and you expect to add
additional nodes to your cluster in the future, then you should generate a
certificate per node so that you can more easily generate new certificates when
you provision new nodes.

######################################################
# 是否需要为每个节点都生成证书，选择 N 无需每个节点都配置证书   #
######################################################
Generate a certificate per node? [y/N]N

## Which hostnames will be used to connect to your nodes?
These hostnames will be added as "DNS" names in the "Subject Alternative Name"
(SAN) field in your certificate.
You should list every hostname and variant that people will use to connect to
your cluster over http.
Do not list IP addresses here, you will be asked to enter them later.

If you wish to use a wildcard certificate (for example *.es.example.com) you
can enter that here.

Enter all the hostnames that you need, one per line.
######################################################
# 输入需连接集群节点主机名信息，一行输入一个IP地址，空行回车结束 #
######################################################
When you are done, press <ENTER> once more to move on to the next step.

es01
es02
es03

You entered the following hostnames.

 - es01
 - es02
 - es03

####################################################
# 确认以上是否为正确的配置，输入 Y 表示信息正确            #
####################################################
Is this correct [Y/n]Y

## Which IP addresses will be used to connect to your nodes?
If your clients will ever connect to your nodes by numeric IP address, then you
can list these as valid IP "Subject Alternative Name" (SAN) fields in your
certificate.

If you do not have fixed IP addresses, or not wish to support direct IP access
to your cluster then you can just press <ENTER> to skip this step.

Enter all the IP addresses that you need, one per line.
####################################################
# 输入需连接集群节点IP信息，一行输入一个IP地址，空行回车结束 #
####################################################
When you are done, press <ENTER> once more to move on to the next step.

172.28.54.213
172.28.54.214
172.28.54.215

You entered the following IP addresses.

 - 172.28.54.213
 - 172.28.54.214
 - 172.28.54.215

####################################################
# 确认以上是否为正确的配置，输入 Y 表示信息正确            #
####################################################
Is this correct [Y/n]Y

## Other certificate options
The generated certificate will have the following additional configuration
values. These values have been selected based on a combination of the
information you have provided above and secure defaults. You should not need to
change these values unless you have specific requirements.

Key Name: es01
Subject DN: CN=es01
Key Size: 2048

####################################################
# 是否要更改以上这些选项，选择 N ，不更改证书选项配置       #
####################################################
Do you wish to change any of these options? [y/N]N

## What password do you want for your private key(s)?

Your private key(s) will be stored in a PKCS#12 keystore file named "http.p12".
This type of keystore is always password protected, but it is possible to use a
blank password.

####################################################
# 是否要给证书加密，不需要加密，两次 回车 即可             #
####################################################
If you wish to use a blank password, simply press <enter> at the prompt below.
Provide a password for the "http.p12" file:  [<ENTER> for none]

## Where should we save the generated files?
A number of files will be generated including your private key(s),
public certificate(s), and sample configuration options for Elastic Stack products.
These files will be included in a single zip archive.
What filename should be used for the output zip file? [/opt/module/elasticsearch-8.3.2/elasticsearch-ssl-http.zip] 
Zip file written to /opt/module/elasticsearch-8.3.2/elasticsearch-ssl-http.zip
```



### 2.6 解压证书并分发

```bash
# 解压
[wangting@es01 bin]$ cd /opt/module/elasticsearch-8.3.2/
[wangting@es01 elasticsearch-8.3.2]$ unzip elasticsearch-ssl-http.zip 
# 移动证书
[wangting@es01 elasticsearch-8.3.2]$ mv ./elasticsearch/http.p12 config/certs/
[wangting@es01 elasticsearch-8.3.2]$ mv ./kibana/elasticsearch-ca.pem config/certs/

# 将证书分发到其他节点02 03
[wangting@es01 elasticsearch-8.3.2]$ cd /opt/module/elasticsearch-8.3.2/config/certs
[wangting@es01 certs]$ ll
total 16
-rw------- 1 wangting wangting 3596 Feb 10 16:05 elastic-certificates.p12
-rw-rw-r-- 1 wangting wangting 1200 Feb 10 16:13 elasticsearch-ca.pem
-rw------- 1 wangting wangting 2672 Feb 10 16:03 elastic-stack-ca.p12
-rw-rw-r-- 1 wangting wangting 3652 Feb 10 16:13 http.p12
[wangting@es01 certs]$ scp * es02:/opt/module/elasticsearch-8.3.2/config/certs/
[wangting@es01 certs]$ scp * es03:/opt/module/elasticsearch-8.3.2/config/certs/
```



> 如果提示：-bash: unzip: command not found，使用yum安装即可
>
> [wangting@es01 elasticsearch-8.3.2]$ sudo yum install -y unzip



### 2.7 配置文件修改配置

```bash
[wangting@es01 certs]$ cd /opt/module/elasticsearch-8.3.2/config/
[wangting@es01 config]$ vim elasticsearch.yml 
cluster.name: bigdata-es
node.name: es-es01
path.data: /opt/module/elasticsearch-8.3.2/data
path.logs: /opt/module/elasticsearch-8.3.2/logs
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["es01"]
cluster.initial_master_nodes: ["es-es01", "es-es02","es-es03"]
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
 enabled: true
 keystore.path: /opt/module/elasticsearch-8.3.2/config/certs/http.p12
 truststore.path: /opt/module/elasticsearch-8.3.2/config/certs/http.p12
xpack.security.transport.ssl:
 enabled: true
 verification_mode: certificate
 keystore.path: /opt/module/elasticsearch-8.3.2/config/certs/elastic-certificates.p12
 truststore.path: /opt/module/elasticsearch-8.3.2/config/certs/elastic-certificates.p12
http.host: [_local_, _site_]
ingest.geoip.downloader.enabled: false
xpack.security.http.ssl.client_authentication: none
```



【注意】：

`xpack.security.http.ssl` & `xpack.security.transport.ssl`后的子配置需要空一格，遵循yml的格式要求

如果不需要后续的http证书认证或者用户密码认证可以将xpack.security相关的功能falase关闭掉



```bash
xpack.security.http.ssl:
 enabled: false
xpack.security.transport.ssl:
 enabled: false
```



有些业务使用场景中，可能会遇到跨域问题，当elasticsearch需要涉及到跨域问题时，可以在配置文件中最后增加配置：

```bash
http.cors.enabled: true
http.cors.allow-origin: "*"
```



### 2.8 修改其余节点的配置文件

```bash
[wangting@es01 elasticsearch-8.3.2]$ scp config/elasticsearch.yml es02:/opt/module/elasticsearch-8.3.2/config/
[wangting@es01 elasticsearch-8.3.2]$ scp config/elasticsearch.yml es03:/opt/module/elasticsearch-8.3.2/config/

# es02修改 config/elasticsearch.yml
[wangting@es02 ~]# vim /opt/module/elasticsearch-8.3.2/config/elasticsearch.yml 
# 设置节点名称
node.name: es-es02

# es03修改 config/elasticsearch.yml
[wangting@es03 ~]# vim /opt/module/elasticsearch-8.3.2/config/elasticsearch.yml 
# 设置节点名称
node.name: es-es03
```





### 2.9 启动集群

每台节点依次启动（无顺序要求，只要多于2台，就可以启动集群，这就是es的无主模式，自动识别集群，选举master）：

```bash
[wangting@es01 elasticsearch-8.3.2]$ /opt/module/elasticsearch-8.3.2/bin/elasticsearch  -d
[wangting@es02 elasticsearch-8.3.2]$ /opt/module/elasticsearch-8.3.2/bin/elasticsearch  -d
[wangting@es03 elasticsearch-8.3.2]$ /opt/module/elasticsearch-8.3.2/bin/elasticsearch  -d
```





> 登录网页，都与之前的密码一致：elastic/bigdata
>
> https://es01:9200/_cat/nodes?v



【注意】

如果不留神使用root修改过目录下文件，则文件权限会变成root所属主，所以需要修改回普通用户



```bash
sudo chown -R wangting.wangting /opt/module/
```



ES服务启动后有2个端口

- 9200为客户端访问es的http协议RESTFUL端口
- 9300为ES集群之间组件的通信端口







### 2-10 修改HTTP登录密码

```bash
# 手工指定elastic的新密码 (-i参数)
[wangting@es01 ~]$ /opt/module/elasticsearch-8.3.2/bin/elasticsearch-reset-password -u elastic -i
warning: ignoring JAVA_HOME=/opt/module/elasticsearch-8.3.2/jdk; using bundled JDK
bThis tool will reset the password of the [elastic] user.
You will be prompted to enter the password.
Please confirm that you would like to continue [y/N]y 
Did not understand answer 'by'
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]: # 输入用户elastic的密码
Re-enter password for [elastic]: # 输入用户elastic的密码
Password for the [elastic] user successfully reset.
```



```
# 也可以不加-i参数让系统随机给个字符串密码，但是很难记住，很少使用
# 为elastic账号自动生成新的随机密码，输出至控制台；不加参数
[wangting@es01 ~]$ /opt/module/elasticsearch-8.3.2/bin/elasticsearch-reset-password -u elastic
```



### 2.11 页面访问验证

> https://ip:9200 (注意是https)
>
> 账号密码为上面创建的：elastic / elastic的密码





### 2.12 集群启动|停止

- 启动服务方式

```bash
[wangting@es01 ~]$ /opt/module/elasticsearch-8.3.2/bin/elasticsearch -d
```



> 【注意】：
>
> 1. -d 为后台运行，不加-d只能前台运行，关了会话窗口服务也会同时终止
> 2. 3台机器都需要启动elasticsearch
> 3. 运行日志没有配置定义，默认在服务目录下：elasticsearch-8.3.2/logs/ ，有异常可以先查看日志



- 停止服务方式

```bash
[wangting@es01 ~]$ ps -ef | grep elasticsearch|grep -vE "grep|controller" |awk -F" " '{print $2}' | xargs kill -9
```



脚本

```bash
# 需先配置免密登录
[wangting@es01 ~]$ ssh-keygen -t rsa
[wangting@es01 ~]$ ssh-copy-id es01
[wangting@es01 ~]$ ssh-copy-id es02
[wangting@es01 ~]$ ssh-copy-id es03

# 编写脚本
[wangting@es01 ~]$ vim my_es.sh
#! /bin/bash
if (($#==0)); then
  echo -e "请输入参数：\n start  启动elasticsearch集群;\n stop  停止elasticsearch集群;\n" && exit
fi


case $1 in
  "start")
    for host in es01 es02 es03
      do
        echo "---------- $1 $host 的elasticsearch ----------"
        ssh $host "/opt/module/elasticsearch-8.3.2/bin/elasticsearch -d >/dev/null 2>&1"
      done
      ;;
  "stop")
    for host in es01 es02 es03
      do
        echo "---------- $1 $host 的elasticsearch ----------"
        ssh $host "ps -ef | grep elasticsearch|grep -v grep|grep -v controller |awk '{print $2}' | xargs kill -9" > /dev/null 2>&1
      done
      ;;
    *)
        echo -e "---------- 请输入正确的参数 ----------\n"
        echo -e "start  启动elasticsearch集群;\n stop  停止elasticsearch集群;\n" && exit
      ;;
esac

```



**启动集群：**

等待脚本执行结束

```shell
[wangting@es01 ~]$ bash my_es.sh start
```

**停止集群：**

```shell
[wangting@es01 ~]$ bash my_es.sh stop
```





---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/elk/elasticsearch-8.10.4%E5%AE%89%E8%A3%85/  

