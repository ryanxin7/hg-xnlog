# Prometheus 简介与安装







## 一、Prometheus 简介
官网：https://prometheus.io/

Prometheus是基于go语言开发的一套开源的监控、报警和时间序列数据库的组合，是由SoundCloud公司开发的开源监控系统，Prometheus于201 6年加入CNCF （Cloud Native Computing Foundation,云原生计算基金会），2018年8月9日prometheus成为CNCF继kubernetes之后毕业的第二二个项目，prometheus在 容器和微服务领域中得到了广泛的应用，其特点主要如下:


```bash
使用key-value的多维度格式保存数据： Prometheus使用标签（labels）来实现多维度的数据存储，允许您更灵活地查询和过滤数据。

使用时序数据库： Prometheus使用时序数据库（TSDB）来存储和查询时间序列数据，这有助于高效地处理大量的指标数据。

支持第三方dashboard： Prometheus可以与第三方仪表板工具集成，例如Grafana，以实现更丰富和可视化的图形界面，提供用户友好的监控仪表板。

组件模块化： Prometheus的组件被设计为模块化的，这使得它易于定制和扩展，同时提高了系统的灵活性。

不依赖传统数据库： Prometheus不使用传统的关系型数据库（如MySQL），而是采用自己的时序数据库，简化了部署和维护过程。

每个采样点仅占3.5字节： Prometheus对数据的高效存储使其能够处理大规模的指标数据，同时节省存储空间。

支持服务自动化发现： Prometheus支持通过诸如Consul等方式进行服务自动发现，使得监控目标的管理更加灵活和自动化。

强大的查询语句功能（PromQL）： Prometheus Query Language（PromQL）提供了强大的查询语言，可以对时间序列数据执行灵活的查询和分析。

数据直接进行算术运算： PromQL允许用户对指标数据进行算术运算，从而更灵活地分析和汇总监控数据。

易于横向伸缩： Prometheus的架构支持横向扩展，可以通过添加更多的实例来处理更多的数据和负载。

众多官方和第三方exporter： Prometheus提供了许多官方和第三方的exporter，这些exporter负责从各种服务和系统中收集指标数据，实现了广泛的数据源覆盖。
```



### 为什么使用 Prometheus？

容器监控的实现方对比虚拟机或者物理机来说比大的区别，比如容器在k8s环境中可以任意横向扩容与缩容，那么就需要监控服务能够自动对新创建的容器进行监控，当容器删除后又能够及时的从监控服务中删除，而传统的zabbix的监控方式需要在每一个容器中安装启动agent， 并且在容器自动发现注册及模板关联方面并没有比较好的实现方式。

Prometheus 更适合监控 Kubernetes，主要是因为以下几个原因：

1. 自动服务发现：Prometheus 可以通过 Kubernetes 的服务发现机制自动发现集群中的服务，不需要手动配置监控目标，这使得在 Kubernetes 中使用 Prometheus 更加方便和易用。

2. 标签化监控：Prometheus 可以通过标签化监控来对 Kubernetes 中的不同组件进行分类和监控，例如可以对同一应用的不同实例进行分类和聚合，并提供对不同实例之间的比较和分析功能。

3. Kubernetes 监控指标：Prometheus 提供了一些特定于 Kubernetes 的监控指标，例如 kubelet、kube-proxy、kube-scheduler 等组件的监控指标，这些指标可以帮助用户更好地了解 Kubernetes 集群和应用程序的健康状况。

4. 容器化：Prometheus 是一个本身就运行在容器中的监控系统，因此它可以很好地与 Kubernetes 的容器化部署模型进行集成。

5. Kubernetes 官方推荐：Kubernetes 官方文档中推荐使用 Prometheus 进行 Kubernetes 监控，并提供了 Prometheus Operator 等工具和库，以帮助用户更轻松地在 Kubernetes 中部署和管理 Prometheus。

   

### Prometheus  组件

```bash
Prometheus Server
#主服务，接受外部HTTP请求,负责收集、存储和查询指标数据。使用PromQL进行数据查询和分析,可以配置告警规则，用于监测和通知异常情况。

Prometheus Targets
#态收集的目标服务数据,Prometheus服务器定期从这些目标中拉取指标数据。

Service Discovery
#于动态发现服务,Prometheus支持多种服务发现机制，例如Consul、Kubernetes、EC2等，使新的目标能够自动加入监控。

Prometheus Alerting
#于配置和管理告警规则。当规则匹配到异常情况时，可以触发告警通知。支持配置多种告警通知方式，如电子邮件、Slack等。

Push Gateway
#据收集代理服务器，类似于Zabbix Proxy的角色。允许短暂的服务（例如批处理作业）将指标推送到Push Gateway，而不需要直接与Prometheus Server通信。对于短暂生命周期的任务，Push Gateway可以更方便地处理指标数据。

Data Visualization and Export
#于数据可视化和导出的组件。可以使用第三方工具如Grafana连接到Prometheus，创建仪表板以实时监视和分析数据。
```





![img](https://cdn1.ryanxin.live/1200756-20220929093158606-1647337583.png)



## 二、部署 Prometheus
可以通过不同的方式安装部署prometheus监控环境，虽然以下的多种安装方式演示了不同的部署方式，但是实际生产环境只需要根据实际需求选择其中一种方式部署即可， 不过无论是使用哪一种方式安装部署的prometheus server，以后的使用都是一样的，后续的课程大部分以二进制安装环境为例，其它会做简单的对应介绍。

- apt 安装  
- docker-ompose 安装 
- 二进制安装





### 2.1 docker-compose部署Prometheus Server

https://github.com/mohamadhoseinmoradi/Docker-Compose-Prometheus-and-Grafana

![image-20240122101201441](https://cdn1.ryanxin.live/image-20240122101201441.png)



安装 docker compose

https://github.com/docker/compose/releases/download/v2.23.3/docker-compose-linux-x86_64



```bash
# 安装好docker后，将项目clone到本地
root@prometheus-server:~# git clone https://github.com/mohamadhoseinmoradi/Docker-Compose-Prometheus-and-Grafana.git
Cloning into 'Docker-Compose-Prometheus-and-Grafana'...
remote: Enumerating objects: 40, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 40 (delta 1), reused 0 (delta 0), pack-reused 32
Unpacking objects: 100% (40/40), 2.90 MiB | 4.47 MiB/s, done.

# 进入目录执行
root@prometheus-server:/apps/docker-compose/Docker-Compose-Prometheus-and-Grafana-master# docker-compose up -d
[+] Running 7/7
 ✔ Container pushgateway   Running                                                                                                                      0.0s
 ✔ Container grafana       Started                                                                                                                      0.5s
 ✔ Container prometheus    Started                                                                                                                      0.6s
 ✔ Container cadvisor      Running                                                                                                                      0.0s
 ✔ Container alertmanager  Running                                                                                                                      0.0s
 ✔ Container nodeexporter  Running                                                                                                                      0.0s
 ✔ Container caddy         Started          
```

![](https://cdn1.ryanxin.live/image-20240122143521141.png)



grafana 账户密码默认是admin/admin

![image-20240122143620417](https://cdn1.ryanxin.live/image-20240122143620417.png)



### 2.2 operator 部署 Prometheus

Operator部署器是基于已经编写好的yaml文件，可以将prometheus server、alertmanager、grafana、 node-exporter等组件一键批量部署。

部署环境：在当前已有的 kubernetes 里部署



![image-20240122143823108](https://cdn1.ryanxin.live/image-20240122143823108.png)



#### 2.2.1 clone 项目并部署

https://github.com/prometheus-operator/kube-prometheus

其中这两个镜像无法下载，需要替换成国内源



**创建命名空间和 CRDs**

```bash
root@k8s-made-01-32:/softs/kube-prometheus-release-0.13# kubectl apply --server-side -f manifests/setup
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheusagents.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/scrapeconfigs.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com serverside-applied
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com serverside-applied
namespace/monitoring serverside-applied


kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
```





```bash
#指定命名空间 "monitoring" 中的 CustomResourceDefinition (CRD) 已经达到了 "Established" 的条件。
#condition met 表示指定的条件已经被满足，资源处于期望的状态
root@k8s-made-01-32:/softs/kube-prometheus-release-0.13# kubectl wait \
> --for condition=Established \
> --all CustomResourceDefinition \
> --namespace=monitoring
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io condition met
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io condition met
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io condition met
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io condition met
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io condition met
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io condition met
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/prometheusagents.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/scrapeconfigs.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com condition met
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com condition met
```



替换镜像

```bash
root@k8s-made-01-32:/softs/kube-prometheus-release-0.13/manifests# grep "registry.k8s.io" ./ -R
./kubeStateMetrics-deployment.yaml:        image: registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.9.2
./prometheusAdapter-deployment.yaml:        image: registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.11.1
```



<br>



```bash
# 执行构建
kubectl apply -f manifests/
```



#### 2.2.2 验证 Pod 状态

![image-20240123153007196](https://cdn1.ryanxin.live/image-20240123153007196.png)

```bash
root@k8s-made-01-32:~# kubectl get pod -n monitoring
NAME                                   READY   STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2     Running   0          20s
blackbox-exporter-59dddb7bb6-582fm     3/3     Running   0          25s
grafana-79f47474f7-r2qb8               1/1     Running   0          24s
kube-state-metrics-744f9b758f-8lrkz    3/3     Running   0          23s
node-exporter-594n4                    2/2     Running   0          23s
node-exporter-5wvgg                    2/2     Running   0          23s
node-exporter-7plwc                    2/2     Running   0          23s
node-exporter-ldzrk                    2/2     Running   0          23s
node-exporter-rjgc6                    2/2     Running   0          23s
prometheus-adapter-69c6c87f9b-m9v2x    1/1     Running   0          22s
prometheus-adapter-69c6c87f9b-qc6t2    1/1     Running   0          22s
prometheus-k8s-0                       2/2     Running   0          18s
prometheus-k8s-1                       2/2     Running   0          18s
prometheus-operator-57cf88fbcb-m2mc7   2/2     Running   0          22s
root@k8s-made-01-32:~# kubectl get statefulsets.apps -n monitoring
NAME                READY   AGE
alertmanager-main   1/1     32s
prometheus-k8s      2/2     30s
```





维护 Prometheus 和 grafana 的配置文件

后期运维主要是维护 Prometheus 和 grafana 的配置文件它们通过 configmap 形式挂载到 kubernetes 里，所以要修改配置就是编辑 configmap

![image-20240123165304463](https://cdn1.ryanxin.live/image-20240123165304463.png)



#### 2.2.3 从外部访问 Prometheus

##### 2.2.3.1 暴露端口

没有暴露端口，所以无法从外部访门 Prometheus

![image-20240123165423326](https://cdn1.ryanxin.live/image-20240123165423326.png)



编辑配置：`/root/kube-prometheus/manifests/prometheus-service.yaml`

![image-20240123170319474](https://cdn1.ryanxin.live/image-20240123170319474.png)

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

![image-20240123165947468](https://cdn1.ryanxin.live/image-20240123165947468.png)



##### 2.2.3.2 修改 NetworkPolicy

上文将端口暴露出来后依然无法从外部访问，那是因为加了 NetworkPolicy ，我们将关于 Prometheus 和 grafana 的 networkpolicy 删除：

![image-20240123170047904](https://cdn1.ryanxin.live/image-20240123170047904.png)

![image-20240123170146894](https://cdn1.ryanxin.live/image-20240123170146894.png)

![image-20240123170402356](https://cdn1.ryanxin.live/image-20240123170402356.png)

之后就能从外部访问了：

![image-20240123170905441](https://cdn1.ryanxin.live/image-20240123170905441.png)



![image-20240123171046830](https://cdn1.ryanxin.live/image-20240123171046830.png)



### 2.3 二进制部署 Prometheus Server



![preview](https://cdn1.ryanxin.live/view)

二进制从官网下载：[Download | Prometheus](https://prometheus.io/download/#prometheus)

#### 2.3.1 解压二进制

https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz

```bash
root@promethues-server:~# tar xf prometheus-2.37.6.linux-amd64.tar.gz
root@promethues-server:~# mkdir /apps
root@promethues-server:~# mv  prometheus-2.37.6.linux-amd64 /apps/
root@promethues-server:/apps# ln -s prometheus-2.37.6.linux-amd64/ /apps/prometheus
root@promethues-server:/apps/prometheus# ll
total 208932
drwxr-xr-x 4 1001  123      4096 Feb 20  2023 ./
drwxr-xr-x 3 root root      4096 Jan 24 09:03 ../
-rw-r--r-- 1 1001  123     11357 Feb 20  2023 LICENSE
-rw-r--r-- 1 1001  123      3773 Feb 20  2023 NOTICE
drwxr-xr-x 2 1001  123      4096 Feb 20  2023 console_libraries/
drwxr-xr-x 2 1001  123      4096 Feb 20  2023 consoles/
-rwxr-xr-x 1 1001  123 111052375 Feb 20  2023 prometheus*      # prometheus服务可执行程序
-rw-r--r-- 1 1001  123       934 Feb 20  2023 prometheus.yml   # prometheus配置文件
-rwxr-xr-x 1 1001  123 102850693 Feb 20  2023 promtool*        


# 测试工具，用于检查配置prometheus配置文件、检测metrics数据等
root@promethues-server:/apps/prometheus# ./promtool check config prometheus.yml
Checking prometheus.yml
 SUCCESS: prometheus.yml is valid prometheus config file syntax
```

#### 2.3.2 创建prometheus service启动脚本

```bash
# 该选项类似 nginx 的 reload，当启用该选项时，Prometheus允许您通过HTTP请求执行优雅关闭或重新加载其配置。
./prometheus --help | grep "enable-lifecycle"
      --web.enable-lifecycle     Enable shutdown and reload via HTTP request.
```

```bash
vim /etc/systemd/system/prometheus.service

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



#### 2.3.3 配置时间同步

```bash
timedatectl set-timezone Asia/Shanghai
apt install chrony
```





#### 2.3.4 启动 prometheus 服务

```bash
root@promethues-server:/apps/prometheus# systemctl restart prometheus.service &&                                                   systemctl enable prometheus.service
Created symlink /etc/systemd/system/multi-user.target.wants/prometheus.service →                                                        /etc/systemd/system/prometheus.service.
```



#### 2.3.5 验证web界面

![image-20240124173131467](https://cdn1.ryanxin.live/image-20240124173131467.png)

#### 2.3.6 动态（热）加载配置

```bash
root@promethues-server:/apps/prometheus# vim /etc/systemd/system/prometheus.service
--web.enable-lifecycle

root@promethues-server:/apps/prometheus# systemctl daemon-reload && systemctl restart prometheus.service

root@promethues-server:/apps/prometheus# curl -X POST http://192.168.29.71:9090/-/reload
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



---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/prometheus/prometheus-%E7%AE%80%E4%BB%8B%E4%B8%8E%E5%AE%89%E8%A3%85/  

