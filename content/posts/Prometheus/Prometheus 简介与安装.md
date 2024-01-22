## Prometheus 简介与安装



## 一、Prometheus 简介
官网：https://prometheus.io/

Prometheus是基于go语言开发的一套开源的监控、报警和时间序列数据库的组合，是由SoundCloud公司开发的开源监控系统，Prometheus于201 6年加入CNCF （Cloud Native Computing Foundation,云原生计算基金会），2018年8月9日prometheus成为CNCF继kubernetes之后毕业的第二二个项目，prometheus在 容器和微服务领域中得到了广泛的应用，其特点主要如下:


```bash
使用key-value的多维度（多个角度，多个层面，多个方面）格式保存数据；

数据不使用MySQL这样的传统数据库，而是使用时序数据库，目前是使用的TSDB；

支持第三方dashboard实现更绚丽的图形界面，如grafana （Grafana 2. 5.0版本及以上）；

组件模块化；

不需要依赖存储，数据可以本地保存也可以远程保存；

平均每个采样点仅占 3.5 bytes，且一个Prometheus server 可以处理数百万级别的的 metrics 指标数据；

支持服务自动化发现（基于 consul 等方式动态发现被监控的目标服务）；

强大的数据查询语句功能（PromQL，Prometheus Query Language）；

数据可以直接进行算术运算；

易于横向伸缩；

众多官方和第三方的 exporter （“数据"导出器）实现不同的指标数据收集。
```



### 为什么使用 Prometheus？

容器监控的实现方对比虚拟机或者物理机来说比大的区别，比如容器在k8s环境中可以任意横向扩容与缩容，那么就需要监控服务能够自动对新创建的容器进行监控，当容器删除后又能够及时的从监控服务中删除，而传统的zabbix的监控方式需要在每一个容器中安装启动agent， 并且在容器自动发现注册及模板关联方面并没有比较好的实现方式。

Prometheus 更适合监控 Kubernetes，主要是因为以下几个原因：

1. 自动服务发现：Prometheus 可以通过 Kubernetes 的服务发现机制自动发现集群中的服务，不需要手动配置监控目标，这使得在 Kubernetes 中使用 Prometheus 更加方便和易用。

2. 标签化监控：Prometheus 可以通过标签化监控来对 Kubernetes 中的不同组件进行分类和监控，例如可以对同一应用的不同实例进行分类和聚合，并提供对不同实例之间的比较和分析功能。

3. Kubernetes 监控指标：Prometheus 提供了一些特定于 Kubernetes 的监控指标，例如 kubelet、kube-proxy、kube-scheduler 等组件的监控指标，这些指标可以帮助用户更好地了解 Kubernetes 集群和应用程序的健康状况。

4. 容器化：Prometheus 是一个本身就运行在容器中的监控系统，因此它可以很好地与 Kubernetes 的容器化部署模型进行集成。

5. Kubernetes 官方推荐：Kubernetes 官方文档中推荐使用 Prometheus 进行 Kubernetes 监控，并提供了 Prometheus Operator 等工具和库，以帮助用户更轻松地在 Kubernetes 中部署和管理 Prometheus。

   

### Prometheus 架构图

几个组件：

```bash
prometheus server: 主服务，接受外部http请求，收集、存储与查询数据等；
prometheus targets: 静态收集的目标服务数据；
service discovery: 动态发现服务；
prometheus alerting: 报警通知；
push gateway: 数据收集代理服务器（类似于zabbix proxy）；
data visualization and export: 数据可视化与数据导出（访问客户端）。
```




![img](https://cdn1.ryanxin.live/6418afe7495bfc33298a6dda7cefe762.png)





## 二、部署 Prometheus
可以通过不同的方式安装部署prometheus监控环境，虽然以下的多种安装方式演示了不同的部署方式，但是实际生产环境只需要根据实际需求选择其中一种方式部署即可， 不过无论是使用哪一种方式安装部署的prometheus server，以后的使用都是一样的，后续的课程大部分以二进制安装环境为例，其它会做简单的对应介绍。

- apt 安装  
- docker-ompose 安装 
- 二进制安装





### 2.1 docker-compose部署Prometheus Server、node-exporter与grafana

https://github.com/mohamadhoseinmoradi/Docker-Compose-Prometheus-and-Grafana

![image-20230302155306464](https://cdn1.ryanxin.live/7f84f915c536a28d738b997529172b0b.png)

```bash
# 安装好docker后，将项目clone到本地
root@prometheus01:~# git clone https://github.com/mohamadhoseinmoradi/Docker-Compose-Prometheus-and-Grafana.git
Cloning into 'Docker-Compose-Prometheus-and-Grafana'...
remote: Enumerating objects: 40, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 40 (delta 1), reused 0 (delta 0), pack-reused 32
Unpacking objects: 100% (40/40), 2.90 MiB | 4.47 MiB/s, done.

# 进入目录执行
root@prometheus01:~# cd Docker-Compose-Prometheus-and-Grafana
root@prometheus01:~/Docker-Compose-Prometheus-and-Grafana# docker-compose up -d
```

修改 `docker-compose.yml` 文件，暴露 Prometheus 的9090端口和 grafana 的3000端口，之后再次执行 `docker-compose up -d`

```dockerfile
 # Prometheus 
    expose:
      - 9090
    ports:
      - "9090:9090"
    networks:
      - monitor-net

# grafana
    expose:
      - 3000
    ports:
      - "3000:3000"
    networks:
      - monitor-net
```

![img](https://cdn1.ryanxin.live/acf3b6780353c4b6e1f311b866a2cc48.png)

grafana 账户密码默认是admin/admin

![img](https://cdn1.ryanxin.live/040e53b5f4d95c33fc1cd9110169245a.png)

### 2.2 operator 部署 Prometheus

Operator部署器是基于已经编写好的yaml文件，可以将prometheus server、alertmanager. grafana、 node-exporter等组件-键批量部署。

部署环境：在当前已有的 kubernetes 里部署

#### 2.2.1 clone 项目并部署

https://github.com/prometheus-operator/kube-prometheus

其中这两个镜像无法下载，需要替换成国内源

![image-20230302165950320](https://cdn1.ryanxin.live/cbada12d59fa32ecae0ddf60c693f6ad.png)

替换镜像源

```bash
root@master01:~/kube-prometheus# vim ./manifests/kubeStateMetrics-deployment.yaml

        image: registry.cn-hangzhou.aliyuncs.com/zhangshijie/kube-state-metrics:2.5.0


root@master01:~/kube-prometheus# vim ./manifests/prometheusAdapter-deployment.yaml

        image: registry.cn-hangzhou.aliyuncs.com/zhangshijie/prometheus-adapter:v0.9.1

```

执行 `kubectl apply -f ./manifests/setup/` 时报错，使用 create 命令执行

![image-20230302171333998](https://cdn1.ryanxin.live/710e47bb0b00526f9bb0cedafce2e033.png)

```bash
root@master01:~/kube-prometheus# kubectl create -f ./manifests/setup/
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
namespace/monitoring created

# 执行构建
root@master01:~/kube-prometheus# kubectl apply -f ./manifests/
```



#### 2.2.2 验证 Pod 状态

![image-20230302202707951](https://cdn1.ryanxin.live/0f42244e31541a2c273cddee94d88ef1.png)

维护 Prometheus 和 grafana 的配置文件

![image-20230302203040462](https://cdn1.ryanxin.live/c63b7b71a2f73b448d2c7491e7db9c4f.png)

#### 2.2.3 从外部访问 Prometheus

##### 2.2.3.1 暴露端口

![image-20230302203313576](https://cdn1.ryanxin.live/06cd7db76352a1501189c47de353e873.png)

编辑这个配置文件：`/root/kube-prometheus/manifests/prometheus-service.yaml`

```yaml
spec:
  type: NodePort #新增
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 39090 # 新增
```

同理，想要从外部访问 grafana ，也要将端口暴露出来，修改这个文件：`/root/kube-prometheus/manifests/grafana-service.yaml`

##### 2.2.3.2 修改 NetworkPolicy

上文将端口暴露出来后依然无法从外部访问，那是因为加了 NetworkPolicy ，我们将关于 Prometheus 和 grafana 的 networkpolicy 删除：

![img](https://cdn1.ryanxin.live/15b0690805263d66177da0060647201b.png)

![img](https://img-blog.csdnimg.cn/img_convert/e206cc8b18175c942489a9410d02aeef.png)

```bash
root@master01:~/kube-prometheus# kubectl delete -f ./manifests/grafana-networkPolicy.yaml 
networkpolicy.networking.k8s.io "grafana" deleted

root@master01:~/kube-prometheus# kubectl delete -f ./manifests/prometheus-networkPolicy.yaml 
networkpolicy.networking.k8s.io "prometheus-k8s" deleted

```

之后就能从外部访问了：

![img](https://cdn1.ryanxin.live/075de11450504db57f22336aa9187609.png)



### 2.3 二进制部署 Prometheus Server

部署环境：本例使用kubernetes 集群外的一台服务器做演示 172.23.1.12

![img](https://cdn1.ryanxin.live/a2eea6189d4286294b81a00223adbe59.png)

二进制从官网下载：[Download | Prometheus](https://prometheus.io/download/#prometheus)

#### 2.3.1 解压二进制

```bash
root@prometheus02:/usr/local/src# pwd
/usr/local/src
root@prometheus02:/usr/local/src# wget https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz
root@prometheus02:/usr/local/src# tar xvf prometheus-2.37.6.linux-amd64.tar.gz 
root@prometheus02:/usr/local/src# mkdir /apps
root@prometheus02:/usr/local/src# mv prometheus-2.37.6.linux-amd64/ /apps/
root@prometheus02:/apps# ln -sv prometheus-2.37.6.linux-amd64/ /apps/prometheus
root@prometheus02:/apps# ll
total 0
drwxr-xr-x  3 root root  61 Mar  2 21:33 ./
drwxr-xr-x 20 root root 304 Mar  2 21:31 ../
lrwxrwxrwx  1 root root  30 Mar  2 21:33 prometheus -> prometheus-2.37.6.linux-amd64//
drwxr-xr-x  4 1001  123 132 Feb 20 18:07 prometheus-2.37.6.linux-amd64/

root@prometheus02:/apps# ll prometheus/
total 208916
drwxr-xr-x 4 1001  123       132 Feb 20 18:07 ./
drwxr-xr-x 3 root root        61 Mar  2 21:33 ../
-rw-r--r-- 1 1001  123     11357 Feb 20 18:03 LICENSE
-rw-r--r-- 1 1001  123      3773 Feb 20 18:03 NOTICE
drwxr-xr-x 2 1001  123        38 Feb 20 18:03 console_libraries/
drwxr-xr-x 2 1001  123       173 Feb 20 18:03 consoles/
-rwxr-xr-x 1 1001  123 111052375 Feb 20 17:40 prometheus*     # prometheus服务可执行程序
-rw-r--r-- 1 1001  123       934 Feb 20 18:03 prometheus.yml  # prometheus配置文件
-rwxr-xr-x 1 1001  123 102850693 Feb 20 17:42 promtool*       # 测试工具，用于检查配置prometheus配置文件、检测metrics数据等

root@prometheus02:/apps/prometheus# ./promtool check config prometheus.yml
Checking prometheus.yml
 SUCCESS: prometheus.yml is valid prometheus config file syntax
```

#### 2.3.2 创建prometheus service启动脚本

```bash
# 该选项类似 nginx 的 reload，修改prometheus配置文件后不用重启服务，直接reload即可
root@prometheus02:/apps/prometheus# ./prometheus --help | grep "enable-lifecycle"
      --web.enable-lifecycle     Enable shutdown and reload via HTTP request.
```

```bash
root@prometheus02:/apps/prometheus# vim /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Restart=on-failure
WorkingDirectory=/apps/prometheus/
ExecStart=/apps/prometheus/prometheus --config.file=/apps/prometheus/prometheus.yml --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```



#### 2.3.3 启动 prometheus 服务

```bash
root@prometheus02:/apps/prometheus# systemctl restart prometheus.service && systemctl enable prometheus.service
```

![image-20230302215033827](https://cdn1.ryanxin.live/c9748de865c16cc7286f404de5a6368c.png)

#### 2.3.4 验证web界面

![image-20230302215200758](https://cdn1.ryanxin.live/a17950a01fb1d76cca276f6fc78453ea.png)

#### 2.3.5 动态（热）加载配置

```bash
root@prometheus02:/apps/prometheus# vim /etc/systemd/system/prometheus.service
--web.enable-lifecycle

root@prometheus02:/apps/prometheus# systemctl daemon-reload && systemctl restart prometheus.service

root@prometheus02:/apps/prometheus# curl -X POST http://172.23.1.12:9090/-/reload

```





### 2.4 二进制安装node-exporter

k8s各node节点使用二进制或者daemonset方式安装node_ exporter，用于收集各k8s node节点宿主机的监控指标数据，默认监听端口为9100。

部署环境：本例部署在kubernetes的两个 node（172.23.0.20/21）节点以及集群外的两个节点（172.23.1.11/13）， 如果之前已经通过其它方式部署了prometheus node- exporter，需要先停止再部署，避免端口冲突。


![image-20230302215912438](https://cdn1.ryanxin.live/1bd077d60b60c3b388451aa3fa3356ea.png)

#### 2.4.1 解压二进制程序

下载地址：[Download | Prometheus](https://prometheus.io/download/#node_exporter)

```bash
root@node01:/usr/local/src# wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz

root@node01:/usr/local/src# tar xvf node_exporter-1.5.0.linux-amd64.tar.gz 
root@node01:/usr/local/src# mkdir /apps
root@node01:/usr/local/src# mv node_exporter-1.5.0.linux-amd64/ /apps/
root@node01:/usr/local/src# cd /apps/
root@node01:/apps# ln -sv node_exporter-1.5.0.linux-amd64/ /apps/node_exporter
'/apps/node_exporter' -> 'node_exporter-1.5.0.linux-amd64/'
root@node01:/apps# ll node_exporter/
total 19336
drwxr-xr-x 2 3434 3434       56 Nov 30 03:05 ./
drwxr-xr-x 4 root root      121 Mar  2 22:02 ../
-rw-r--r-- 1 3434 3434    11357 Nov 30 03:05 LICENSE
-rw-r--r-- 1 3434 3434      463 Nov 30 03:05 NOTICE
-rwxr-xr-x 1 3434 3434 19779640 Nov 30 02:59 node_exporter*
```





#### 2.4.2 创建node-exporter service启动文件

```bash
root@node01:/apps# vim /etc/systemd/system/node-exporter.service

[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
ExecStart=/apps/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
```



#### 2.4.3 启动node exporter服务

```bash
# 默认监听在9100端口
root@node01:/apps# ./node_exporter/node_exporter --help | grep 9100
      --web.listen-address=:9100 ...  
```

```bash
root@node01:/apps# systemctl daemon-reload && systemctl restart node-exporter.service && systemctl enable node-exporter.service 
```



#### 2.4.4 验证web页面

![image-20230302222317782](https://cdn1.ryanxin.live/e944b2af5a9a4d1f56cd74228f5884aa.png)





#### 2.4.5 node-exporter指标数据
[4.5.2. kubernetes-cadvisor — 新溪-gordon V1.7.0 documentation (zhaoweiguo.com)](https://knowledge.zhaoweiguo.com/build/html/cloudnative/prometheus/metrics/kubernetes-cadvisor.html)

```bash
root@prometheus02:/apps# curl 172.23.0.20:9100/metrics

常见指标：
node_boot_time: 系统自启动以后的总结时间
node_cpu：系统CPU使用量
node_disk*：磁盘IO
node_filesystem*: 系统文件系统用量
node_load1: 系统CPU负载
node_memeory*: 内存使用量
node_network*:网络带宽指标
node_time: 当前系统时间
go_*: node exporter中go相关指标
process_ *: node exporter 自身进程相关运行指标
```

至此node节点（172.23.0.20）已经安装了node-exporter，将另外一个node节点（172.23.0.21）也安装node-exporter。



### 2.5 配置prometheus server收集node-exporter指标数据

上文部署好prometheus server后，它只收集了自身的指标数据，那么怎么让它也收集node-exporter指标数据？

![image-20230302224307106](https://cdn1.ryanxin.live/6fb1a4e7ce16f3f58fa44b15469d54c0.png)

#### 2.5.1 prometheus 默认配置文件

```bash
root@prometheus02:/apps# vim /apps/prometheus/prometheus.yml 

# my global config
global:
  scrape_interval: 15s # 数据收集间隔时间，如果不配置默认为一分钟
  evaluation_interval: 15s # 规则扫描间隔时间，如果不配置默认为一分钟
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:  # 报警通知配置
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:  # 规则配置
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:  # 数据采集目标配置
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```

#### 2.5.2 添加 node节点数据收集

```bash
root@prometheus02:/apps# vim /apps/prometheus/prometheus.yml 
# 末尾添加
  - job_name: "prometheus-worknode"
    static_configs:
      - targets: ["172.23.0.20:9100","172.23.0.21:9100"]
```

#### 2.5.3 动态加载配置并验证prometheus server状态

```bash
root@prometheus02:/apps# curl -X POST http://172.23.1.12:9090/-/reload
```

![image-20230302225615678](https://cdn1.ryanxin.live/2a07515b6df5e5d97c642c49c3b8dcff.png)



### 2.6 部署 Grafana
grafana是一个可视化组件，用于接收客户端浏览器的请求并连接到prometheus查询数据，最后经过渲染并在浏览器进行体系化显示，需要注意的是，grafana查询数据类似于zabbix-样需要自定义模板,模板可以手动制作也可以导入已有模板。

下载：[Download Grafana | Grafana Labs](https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1)

模板：[Dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/)

插件：[Grafana Plugins - extend and customize your Grafana | Grafana Labs](https://grafana.com/grafana/plugins/)


![image-20230303115811593](https://cdn1.ryanxin.live/8d82119f52d9bdb6f5a8da7242918ba2.png)

#### 2.6.1 安装 grafana server

下载地址：[Download Grafana | Grafana Labs](https://grafana.com/grafana/download)

安装文档：[Install Grafana | Grafana documentation](https://grafana.com/docs/grafana/latest/setup-grafana/installation/)

部署环境：可以和 Prometheus Server 安装在一起，也可以分开安装（网络互通即可）。本例和 Prometheus Server 安装在一起（172.23.1.12）。


```bash
root@prometheus-server:~# sudo apt-get install -y adduser libfontconfig1
root@prometheus-server:~# wget https://dl.grafana.com/enterprise/release/grafana-enterprise_9.4.3_amd64.deb
root@prometheus-server:~# sudo dpkg -i grafana-enterprise_9.4.3_amd64.deb
```

#### 2.6.2 grafana server 配置文件

路径：`/etc/grafana/grafana.ini`

```bash
[server]
# Protocol (http, https, h2, socket)
protocol = http

# The ip address to bind to, empty will bind to all interfaces
http_addr = 0.0.0.0

# The http port  to use
http_port = 3000
```

#### 2.6.3 启动 grafana

```bash
root@prometheus-server:~# systemctl restart grafana-server.service && systemctl enable grafana-server.service 
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /lib/systemd/system/grafana-server.service.
root@prometheus-server:~# ss -anptl | grep 3000
LISTEN    0         4096                     *:3000                   *:*        users:(("grafana",pid=136786,fd=11))                                           
```



#### 2.6.4 验证 web 界面

默认账户密码：admin/admin

![image-20230303121328026](https://cdn1.ryanxin.live/04414deb26df1587a0eb779899cbaa1e.png)

#### 2.6.5 添加 Prometheus 数据源

进入主界面后，点击左下角的设置，选择 “ Data sources”，再选择 Prometheus。

![image-20230303121741323](https://cdn1.ryanxin.live/1ec32342b6075cb8cc4a79741b5a3d29.png)

![image-20230303122140995](https://cdn1.ryanxin.live/3e29221f0db6c59f28ec8723b7b8ea61.png)

![image-20230303122254292](https://cdn1.ryanxin.live/57139d1900b90f0cb6dcc6665e2f7c3a.png)



#### 2.6.6 导入模板

模板仓库：[Dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/)

推荐使用：[1 Node Exporter for Prometheus Dashboard EN 20201010 | Grafana Labs](https://grafana.com/grafana/dashboards/11074-node-exporter-for-prometheus-dashboard-en-v20201010/)



![image-20230303123010063](https://cdn1.ryanxin.live/f57274e5b4ce40883d158b5d674d97b6.png)

点击 Import

![image-20230303123250724](https://cdn1.ryanxin.live/d1abec5f15c90555a6cfe1145178cc5e.png)

输入模板ID

![image-20230303123559425](https://cdn1.ryanxin.live/17f40802c97a4dff824f75b127c339c7.png)

选择数据源

![image-20230303123711522](https://cdn1.ryanxin.live/82f4b0929be7df89e5fd12402f16f34a.png)

验证模板图形信息

![image-20230303123815999](https://cdn1.ryanxin.live/f34cbb71bd8d0a03b7ef306170065f60.png)

6、插件管理
插件仓库：[Grafana Plugins - extend and customize your Grafana | Grafana Labs](https://grafana.com/grafana/plugins/)

本例安装饼图插件：[Pie Chart plugin for Grafana | Grafana Labs]()

插件保存目录：`/var/lib/grafana/plugins`

```bash
# 在线安装
root@prometheus-server:~# grafana-cli plugins install grafana-piechart-panel
✔ Downloaded and extracted grafana-piechart-panel v1.6.4 zip successfully to /var/lib/grafana/plugins/grafana-piechart-panel

Please restart Grafana after installing plugins. Refer to Grafana documentation for instructions if necessary.
```



```bash
# 离线安装
wget -nv https://grafana.com/api/plugins/grafana-piechart-panel/versions/latest/download -O /tmp/grafana-piechart-panel.zip
unzip -q /tmp/grafana-piechart-panel.zip -d /tmp
mv /tmp/grafana-piechart-panel-* /var/lib/grafana/plugins/grafana-piechart-panel
sudo service grafana-server restart
```

```bash
# 您可以将此repo直接克隆到插件目录中。然后重新启动grafana服务器，插件将被自动检测并使用。
git clone https://github.com/grafana/piechart-panel.git --branch release-1.6.2
sudo service grafana-server restart
```

![image-20230303143432271](https://cdn1.ryanxin.live/6996b9be22c66525cd2f1f47cf5db669.png)



## 三、PromQL 语句
官网：[Querying basics | Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)

Prometheus 提供一个函数式的表达式语言 PromQL（Prometheus Query Language），可以使用户实时地查找和聚合时间序列数据，表达式计算结果可以在图表中展示，也可以在Prometheus浏览器中以表格形式展示，或者作为数据源，以 HTTP API 的方式提供给外部系统使用。


### 3.1 PromQL 数据基础

#### 3.1.1 PromQL查询数据类型

https://prometheus.io/docs/prometheus/latest/querying/basics/

**瞬时向量、瞬时数据（instant vector）**：是对目标实例查询到的同一个时间戳的一组时间序列数据（按照时间的推移对数据进存储和展示），每个时间序列包含单个数据样本，比如 node_memory_MemFree_bytes 查询的是当前剩余内存（可用内存）就是-个瞬时向量，该表达式的返回值中只会包含该时间序列中的最新的一个样本值，而相应的这样的表达式称之为瞬时向量表达式，例如：`prometheus_http_requests_total`，prometheus API 查询瞬时数据命令，在没有指定匹配条件的前提下，会返回所有包含此指标数据的实例数据:

```bash
curl 'http://172.23.1.12:9090/api/v1/query' --data 'query=node_memory_MemFree_bytes' --data time=1677826800
```

![img](https://cdn1.ryanxin.live/0bb0f13c85caec97bc91fd6f338d4d9a.png)

**标量、纯量数据（scalar）**：是一个浮点数类型的数据值，使用 node_ load1 获取到时一个瞬时向量后，再使用 prometheus 的内置函数 scalar( ) 将瞬时向量转换为标量，例如：`scalar(sum(node_load1))`

```bash
curl 'http://172.23.1.12:9090/api/v1/query' --data 'query=scalar(sum(node_load1{instance="172.23.1.11:9100"}))' --data time=1677826800
```

