---
author: Ryan
title: K8S集群证书过期处理-Node NoReady状态
date: 2023-09-13
lastmod: 2023-09-13
tags:
  - Kubernetes
categories:
  - Kubernetes
expirationReminder:
  enable: false
---





---





{{< admonition type=warning  open=true >}}

Kubelet 客户端证书在 Kubernetes 集群中扮演着重要的角色，因为它们用于节点与控制平面之间的安全通信和身份验证。当 kubelet 客户端证书在主节点和工作节点之间不一致时，可能会导致以下问题：



1. **认证问题**：如果主节点和工作节点上的 kubelet 客户端证书不匹配，工作节点可能无法成功通过 kube-apiserver 连接到集群控制平面。这可能会导致节点在集群中无法注册，或者在节点重新启动后无法重新加入集群。

   

2. **授权问题**：证书不匹配可能导致工作节点无法获得所需的授权来执行其任务。这可能会影响到节点上运行的 Pod 的权限，导致访问受限或拒绝访问控制。

3. **安全性问题**：证书不一致可能会降低集群的整体安全性，因为节点身份的验证不再可靠。攻击者可能会利用此漏洞来伪装为不同的节点，试图执行恶意操作或访问敏感数据。

4. **集群稳定性问题**：证书问题可能导致节点的异常行为，例如节点的意外停机或无法访问。这可能会导致应用程序中断和集群不稳定

{{< /admonition >}}



## 1. 排查过程


查看pod状态发现 `zz-biz-01`节点上的pod全部为 **Terminating** 状态

![dbc44720c0482997709674bb501eb98](https://cdn1.ryanxin.live/dbc44720c0482997709674bb501eb98.png)





查看集群节点状态 zz-biz-01 为**NoReady** 状态

```
[root@zz-front-01 ~]# kubectl get node
NAME          STATUS   ROLES                  AGE    VERSION
zz-biz-01     NoReady    <none>                 729d   v1.21.4
zz-front-01   Ready    control-plane,master   729d   v1.21.4
```



查看该节点详细信息:



在Conditions中显示**NodeStatusUnknown**，说明该节点与master节点通信异常，可能是网络问题也可能是证书认证问题。

```bash
[root@zz-front-01 pki]# kubectl describe node zz-biz-01
Name:               zz-biz-01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=zz-biz-01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"9e:4b:af:c8:84:39"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 172.21.10.158
                    kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 13 Sep 2021 20:43:32 +0800
Taints:             node.kubernetes.io/unreachable:NoExecute
                    node.kubernetes.io/unreachable:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  zz-biz-01
  AcquireTime:     <unset>
  RenewTime:       Wed, 17 May 2023 12:11:16 +0800
Conditions:
  Type                 Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----                 ------    -----------------                 ------------------                ------              -------
  NetworkUnavailable   False     Sat, 22 Apr 2023 13:18:44 +0800   Sat, 22 Apr 2023 13:18:44 +0800   FlannelIsUp         Flannel is running on thi                      s node
  MemoryPressure       Unknown   Wed, 17 May 2023 12:10:38 +0800   Tue, 12 Sep 2023 11:07:27 +0800   NodeStatusUnknown   Kubelet stopped posting n                      ode status.
  DiskPressure         Unknown   Wed, 17 May 2023 12:10:38 +0800   Tue, 12 Sep 2023 11:07:27 +0800   NodeStatusUnknown   Kubelet stopped posting n                      ode status.
  PIDPressure          Unknown   Wed, 17 May 2023 12:10:38 +0800   Tue, 12 Sep 2023 11:07:27 +0800   NodeStatusUnknown   Kubelet stopped posting n                      ode status.
  Ready                Unknown   Wed, 17 May 2023 12:10:38 +0800   Tue, 12 Sep 2023 11:07:27 +0800   NodeStatusUnknown   Kubelet stopped posting n                      ode status.
Addresses:
  InternalIP:  172.21.10.158
  Hostname:    zz-biz-01
Capacity:
  cpu:                8
  ephemeral-storage:  51175Mi
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16265960Ki
  pods:               110
Allocatable:
  cpu:                8
  ephemeral-storage:  48294789041
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16163560Ki
  pods:               110
System Info:
  Machine ID:                 9f225d6ba3fa4e24acb9d493fcc2a126
  System UUID:                420B4078-BE8A-D926-2521-CFE406B4A8B9
  Boot ID:                    14618e98-5f60-4483-8afe-dffe3ecb1319
  Kernel Version:             3.10.0-1160.42.2.el7.x86_64
  OS Image:                   CentOS Linux 7 (Core)
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.4.9
  Kubelet Version:            v1.21.4
  Kube-Proxy Version:         v1.21.4
PodCIDR:                      10.244.1.0/24
PodCIDRs:                     10.244.1.0/24
Non-terminated Pods:          (2 in total)
  Namespace                   Name                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                     ------------  ----------  ---------------  -------------  ---
  kube-system                 kube-flannel-ds-lk647    100m (1%)     100m (1%)   50Mi (0%)        50Mi (0%)      728d
  kube-system                 kube-proxy-vgd4p         0 (0%)        0 (0%)      0 (0%)           0 (0%)         728d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (1%)  100m (1%)
  memory             50Mi (0%)  50Mi (0%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:
  Type     Reason                   Age                     From     Message
  ----     ------                   ----                    ----     -------
  Normal   Starting                 60m                     kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      60m                     kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  60m                     kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasSufficientMemory  60m (x5 over 60m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    60m (x5 over 60m)       kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     60m (x5 over 60m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Warning  InvalidDiskCapacity      59m                     kubelet  invalid capacity 0 on image filesystem
  Normal   Starting                 59m                     kubelet  Starting kubelet.
  Normal   NodeAllocatableEnforced  59m                     kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    59m (x7 over 59m)       kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     59m (x7 over 59m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  49m (x86 over 59m)      kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   Starting                 49m                     kubelet  Starting kubelet.
  Normal   Starting                 49m                     kubelet  Starting kubelet.
  Normal   Starting                 48m                     kubelet  Starting kubelet.
  Normal   Starting                 47m                     kubelet  Starting kubelet.
  Normal   Starting                 46m                     kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      46m                     kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  46m                     kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasSufficientMemory  46m (x3 over 46m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    46m (x3 over 46m)       kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     46m (x3 over 46m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   Starting                 46m                     kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      46m                     kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  45m                     kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    45m (x7 over 46m)       kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     45m (x7 over 46m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  15m (x253 over 46m)     kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   Starting                 14m                     kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      14m                     kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  14m                     kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    13m (x7 over 14m)       kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     13m (x7 over 14m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  13m (x8 over 14m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   Starting                 12m                     kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      12m                     kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  12m                     kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    11m (x7 over 12m)       kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     11m (x7 over 12m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  11m (x8 over 12m)       kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   Starting                 9m55s                   kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      9m55s                   kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  9m55s                   kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    9m42s (x7 over 9m55s)   kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     9m42s (x7 over 9m55s)   kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  4m53s (x47 over 9m55s)  kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
  Normal   Starting                 3m20s                   kubelet  Starting kubelet.
  Warning  InvalidDiskCapacity      3m20s                   kubelet  invalid capacity 0 on image filesystem
  Normal   NodeAllocatableEnforced  3m20s                   kubelet  Updated Node Allocatable limit across pods
  Normal   NodeHasNoDiskPressure    3m7s (x7 over 3m20s)    kubelet  Node zz-biz-01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     3m7s (x7 over 3m20s)    kubelet  Node zz-biz-01 status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  3m (x8 over 3m20s)      kubelet  Node zz-biz-01 status is now: NodeHasSufficientMemory
```



查看master节点kubelet日志信息



```bash
[root@zz-front-01 ~]# systemctl status kubelet.service -l
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since 六 2023-04-22 13:16:49 CST; 4 months 21 days ago
     Docs: https://kubernetes.io/docs/
 Main PID: 1202 (kubelet)
    Tasks: 27
   Memory: 151.1M
   CGroup: /system.slice/kubelet.service
           └─1202 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubele     t/config.yaml --container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock --pod-infra-container-image=harbor.rmxc.tech/k8s-gcr-i     mages/pause:3.4.1

9月 12 10:31:36 zz-front-01 kubelet[1202]: E0912 10:31:36.403864    1202 nestedpendingoperations.go:301] Operation for "{volumeName:kubernetes.io/projected/f1b313c     c-311f-4ab9-8aee-da22ca5416a2-kube-api-access-82qbr podName:f1b313cc-311f-4ab9-8aee-da22ca5416a2 nodeName:}" failed. No retries permitted until 2023-09-12 10:33:38     .403807746 +0800 CST m=+12345406.211185998 (durationBeforeRetry 2m2s). Error: "MountVolume.SetUp failed for volume \"kube-api-access-82qbr\" (UniqueName: \"kuberne     tes.io/projected/f1b313cc-311f-4ab9-8aee-da22ca5416a2-kube-api-access-82qbr\") pod \"ingress-nginx-controller-5687cff78c-cfvbt\" (UID: \"f1b313cc-311f-4ab9-8aee-da     22ca5416a2\") : failed to fetch token: Post \"https://zz-front-01:6443/api/v1/namespaces/ingress-nginx/serviceaccounts/ingress-nginx/token\": dial tcp 172.21.10.15     7:6443: connect: connection refused"
9月 12 10:31:38 zz-front-01 kubelet[1202]: E0912 10:31:38.394737    1202 reflector.go:138] object-"ingress-nginx"/"ingress-nginx-admission": Failed to watch *v1.Se     cret: failed to list *v1.Secret: Get "https://zz-front-01:6443/api/v1/namespaces/ingress-nginx/secrets?fieldSelector=metadata.name%3Dingress-nginx-admission&resour     ceVersion=132547327": dial tcp 172.21.10.157:6443: connect: connection refused
9月 12 10:31:39 zz-front-01 kubelet[1202]: I0912 10:31:39.161512    1202 status_manager.go:566] "Failed to get status for pod" podUID=887f7c42-fe79-4471-8cdc-4d94f     894d4c5 pod="zz-prod/fe-nginx-84fbf5cbcc-zvmsf" error="Get \"https://zz-front-01:6443/api/v1/namespaces/zz-prod/pods/fe-nginx-84fbf5cbcc-zvmsf\": dial tcp 172.21.1     0.157:6443: connect: connection refused"
9月 12 10:31:39 zz-front-01 kubelet[1202]: I0912 10:31:39.161970    1202 status_manager.go:566] "Failed to get status for pod" podUID=f1b313cc-311f-4ab9-8aee-da22c     a5416a2 pod="ingress-nginx/ingress-nginx-controller-5687cff78c-cfvbt" error="Get \"https://zz-front-01:6443/api/v1/namespaces/ingress-nginx/pods/ingress-nginx-cont     roller-5687cff78c-cfvbt\": dial tcp 172.21.10.157:6443: connect: connection refused"
9月 12 10:31:39 zz-front-01 kubelet[1202]: I0912 10:31:39.162338    1202 status_manager.go:566] "Failed to get status for pod" podUID=b68b9cc1-df20-44e5-ad09-3e06b     ad98bb3 pod="zz-prod/mysql57-5d46dc4786-x9nb7" error="Get \"https://zz-front-01:6443/api/v1/namespaces/zz-prod/pods/mysql57-5d46dc4786-x9nb7\": dial tcp 172.21.10.     157:6443: connect: connection refused"
9月 12 10:31:39 zz-front-01 kubelet[1202]: I0912 10:31:39.162601    1202 status_manager.go:566] "Failed to get status for pod" podUID=c053119073c29f2af21835ffd25ec     d51 pod="kube-system/kube-apiserver-zz-front-01" error="Get \"https://zz-front-01:6443/api/v1/namespaces/kube-system/pods/kube-apiserver-zz-front-01\": dial tcp 17     2.21.10.157:6443: connect: connection refused"
9月 12 10:31:39 zz-front-01 kubelet[1202]: I0912 10:31:39.162818    1202 status_manager.go:566] "Failed to get status for pod" podUID=844359d0-0d59-4e4d-8014-12c5d     930c9f8 pod="kube-system/nfs-client-nfs-subdir-external-provisioner-7b4dcd59f8-79mk8" error="Get \"https://zz-front-01:6443/api/v1/namespaces/kube-system/pods/nfs-     client-nfs-subdir-external-provisioner-7b4dcd59f8-79mk8\": dial tcp 172.21.10.157:6443: connect: connection refused"
9月 12 10:31:40 zz-front-01 kubelet[1202]: E0912 10:31:40.834602    1202 reflector.go:138] k8s.io/client-go/informers/factory.go:134: Failed to watch *v1.Node: fai     led to list *v1.Node: Get "https://zz-front-01:6443/api/v1/nodes?fieldSelector=metadata.name%3Dzz-front-01&resourceVersion=132547450": dial tcp 172.21.10.157:6443:      connect: connection refused
9月 12 10:31:41 zz-front-01 kubelet[1202]: I0912 10:31:41.161636    1202 scope.go:111] "RemoveContainer" containerID="9de69e79721d6ab0ee407dfcd8bd16eba150b941175b2     c1b9606c2dbd4ab4940"
9月 12 10:31:41 zz-front-01 kubelet[1202]: E0912 10:31:41.162682    1202 pod_workers.go:190] "Error syncing pod, skipping" err="failed to \"StartContainer\" for \"     controller\" with CrashLoopBackOff: \"back-off 5m0s restarting failed container=controller pod=ingress-nginx-controller-5687cff78c-cfvbt_ingress-nginx(f1b313cc-311     f-4ab9-8aee-da22ca5416a2)\"" pod="ingress-nginx/ingress-nginx-controller-5687cff78c-cfvbt" podUID=f1b313cc-311f-4ab9-8aee-da22ca5416a2
```



这些日志条目是来自一个名为 "zz-front-01" 的节点上运行的 Kubernetes kubelet 的记录。这些日志指示 kubelet 与 Kubernetes API 服务器的连接存在一些问题，kubelet 在连接到 API 服务器时遇到问题，导致出现各种错误。

以下是日志中提到的主要问题的解释：

1. **连接被拒绝到 API 服务器**：kubelet 报告无法连接到 "[https://zz-front-01:6443](https://zz-front-01:6443/)" 上的 Kubernetes API 服务器。这由错误消息 "dial tcp 172.21.10.157:6443: connect: connection refused" 表示。kubelet 依赖 API 服务器执行各种任务，所以这个连接问题会导致各种问题。
2. **无法监视资源**：kubelet 无法监视和列出来自 API 服务器的资源，如 Secrets 和 Nodes。这个失败很可能与上述连接问题有关。它影响了各种组件，例如 Ingress 控制器、MySQL pods、kube-apiserver 等。
3. **无法获取 Pod 的状态**：kubelet 也无法获取来自 API 服务器的 pod 状态。这会影响不同命名空间中的各种 pod，包括 "zz-prod/fe-nginx-84fbf5cbcc-zvmsf"、"ingress-nginx/ingress-nginx-controller-5687cff78c-cfvbt"、"zz-prod/mysql57-5d46dc4786-x9nb7"、"kube-system/kube-apiserver-zz-front-01" 等等。
4. **同步 Pod 时出错**：在尝试同步 "ingress-nginx-controller-5687cff78c-cfvbt" pod 时报告了一个错误。错误提到了 "CrashLoopBackOff" 条件，通常表示 pod 反复启动失败。

这些问题的根本原因似乎是 kubelet 无法建立与运行在 "zz-front-01:6443" 上的 Kubernetes API 服务器的连接。





检查集群证书

可以看到已经过期了

![image-20230912104901164](https://cdn1.ryanxin.live/image-20230912104901164.png)

![image-20230912105524775](https://cdn1.ryanxin.live/image-20230912105524775.png)



## 2.处理过程

刷新master证书，过程参考
https://www.xinn.cc/posts/notes/k8s-certificate-missery/ 
https://www.jianshu.com/p/10269275239d



通过查看`/etc/kubernetes/kubelet.conf` 发现证书路径`/var/lib/kubelet/pki/kubelet-client-current.pem`



到`/var/lib/kubelet/pki/` 路径下查看证书日期

```bash
[root@zz-biz-01 pki]# ls
apiserver-kubelet-client.crt  kubelet-client-2021-09-13-20-43-21.pem  kubelet-client-current.pem  kubelet.key
apiserver-kubelet-client.key  kubelet-client-2022-08-16-09-23-17.pem  kubelet.crt                 kubelet.key.bak
bak                           kubelet-client-2023-04-25-14-58-57.pem  kubelet.crt.bak
```



查看证书有效期

```bash
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -text | grep Not
```





批量清除`zz-biz-01`节点上全部为 **Terminating** 状态的Pod

```sh
kubectl get pods -n kube-system | grep Terminating | awk '{print $1}' | xargs kubectl delete pod -n kube-system --force --grace-period=0

kubectl get pods -n zz-prod | grep Terminating | awk '{print $1}' | xargs kubectl delete pod -n zz-prod --force --grace-period=0
```





查看worker 节点的kubelet日志

证书已经更新了但是worker节点还是出现认证问题：

```bash
9月 13 09:21:36 zz-biz-01 systemd[1]: Unit kubelet.service entered failed state.
9月 13 09:21:36 zz-biz-01 systemd[1]: kubelet.service failed.
9月 13 09:21:47 zz-biz-01 systemd[1]: kubelet.service holdoff time over, scheduling restart.
9月 13 09:21:47 zz-biz-01 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
9月 13 09:21:47 zz-biz-01 systemd[1]: Started kubelet: The Kubernetes Node Agent.
9月 13 09:21:47 zz-biz-01 kubelet[14097]: I0913 09:21:47.127825   14097 server.go:197] "Warning: For remote container runtime, --pod-infra-contain                      er-image is ignored in kubelet, which should be set in that remote runtime instead"
9月 13 09:21:47 zz-biz-01 kubelet[14097]: I0913 09:21:47.155032   14097 server.go:440] "Kubelet version" kubeletVersion="v1.21.4"
9月 13 09:21:47 zz-biz-01 kubelet[14097]: I0913 09:21:47.155367   14097 server.go:851] "Client rotation is on, will bootstrap in background"
9月 13 09:21:47 zz-biz-01 kubelet[14097]: E0913 09:21:47.158797   14097 bootstrap.go:240] unable to read existing bootstrap client config from /et                      c/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due                       to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client                      -current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
9月 13 09:21:47 zz-biz-01 systemd[1]: kubelet.service: main process exited, code=exited, status=1/FAILURE
9月 13 09:21:47 zz-biz-01 kubelet[14097]: E0913 09:21:47.158910   14097 server.go:292] "Failed to run kubelet" err="failed to run Kubelet: unable                       to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
9月 13 09:21:47 zz-biz-01 systemd[1]: Unit kubelet.service entered failed state.
9月 13 09:21:47 zz-biz-01 systemd[1]: kubelet.service failed.
9月 13 09:21:57 zz-biz-01 systemd[1]: kubelet.service holdoff time over, scheduling restart.
9月 13 09:21:57 zz-biz-01 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
9月 13 09:21:57 zz-biz-01 systemd[1]: Started kubelet: The Kubernetes Node Agent.
9月 13 09:21:57 zz-biz-01 kubelet[14136]: I0913 09:21:57.395574   14136 server.go:197] "Warning: For remote container runtime, --pod-infra-contain                      er-image is ignored in kubelet, which should be set in that remote runtime instead"
9月 13 09:21:57 zz-biz-01 kubelet[14136]: I0913 09:21:57.413917   14136 server.go:440] "Kubelet version" kubeletVersion="v1.21.4"
9月 13 09:21:57 zz-biz-01 kubelet[14136]: I0913 09:21:57.414228   14136 server.go:851] "Client rotation is on, will bootstrap in background"
9月 13 09:21:57 zz-biz-01 kubelet[14136]: E0913 09:21:57.415375   14136 bootstrap.go:240] unable to read existing bootstrap client config from /et                      c/kubernetes/kubelet.conf: invalid configuration: [unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due                       to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory, unable to read client-key /var/lib/kubelet/pki/kubelet-client                      -current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: no such file or directory]
9月 13 09:21:57 zz-biz-01 systemd[1]: kubelet.service: main process exited, code=exited, status=1/FAILURE
9月 13 09:21:57 zz-biz-01 kubelet[14136]: E0913 09:21:57.415411   14136 server.go:292] "Failed to run kubelet" err="failed to run Kubelet: unable                       to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory"
9月 13 09:21:57 zz-biz-01 systemd[1]: Unit kubelet.service entered failed state.
9月 13 09:21:57 zz-biz-01 systemd[1]: kubelet.service failed.
```





**解决方法**：在master节点证书更新完成后，获取加入集群token，重新将worker节点加入集群



**注意**: woker节点添加进集群（需删除原先kubelet配置文件，否则加入失败）

先备份下配置文件的存放目录

```bash
#备份文件
cp -r /etc/kubernetes /etc/kubernetes.bak
```



```bash
#删除kubelet配置文件并停止kubelet服务
[root@zz-biz-01 kubernetes]# rm -rf /etc/kubernetes/kubelet.conf
[root@zz-biz-01 kubernetes]# rm -rf /etc/kubernetes/pki/ca.crt
[root@zz-biz-01 kubernetes]# systemctl stop kubelet.service
```



```bash
#woker节点重新加入集群

[root@zz-biz-01 kubernetes]# kubeadm join zz-front-01:6443 --token 4ekwze.3et9dyulz179u68x --discovery-token-ca-cert-hash sha256:f33bd69abd60e75ebebdd28526c535e6e93acf3b80a45629dfe5be59cb38e1bc
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```





## 3.验证集群状态

```bash
[root@zz-front-01 ~]# kubectl get node
NAME          STATUS   ROLES                  AGE    VERSION
zz-biz-01     Ready    <none>                 729d   v1.21.4
zz-front-01   Ready    control-plane,master   729d   v1.21.4
```



![image-20230913163810231](https://cdn1.ryanxin.live/image-20230913163810231.png)

![image-20230913164046844](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20230913164046844.png)

![image-20230913163837544](https://cdn1.ryanxin.live/image-20230913163837544.png)













