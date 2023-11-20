---
author: Ryan
title: 基于K8S 动态持久化存储部署Alist项目
date: 2023-11-20
lastmod: 2023-11-20
tags:
  - Kubernetes实战案例
  - AList
categories:
  - Kubernetes
expirationReminder:
  enable: false
---



## 基于K8S 动态持久化存储 部署Alist项目

AList 支持多个存储提供商，包括本地存储、阿里云盘、OneDrive、Google Drive 等，且易于拓展。

之前使用**docker-compose**方式部署，这次试试基于K8S并使用持久化存储卷作为后端存储部署。

![image-20231120170556658](https://cdn1.ryanxin.live/image-20231120170556658.png)

项目地址：https://github.com/alist-org/alist

使用指南：https://alist.nn.ci/zh/



## 创建 Deployment 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alist-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alist
  template:
    metadata:
      labels:
        app: alist
    spec:
      containers:
        - name: alist
          image: harbor.ceamg.com/baseimages/alist:xxnn0.1
          ports:
            - containerPort: 5244
          volumeMounts:
            - name: alist-data-volume
              mountPath: /opt/alist/data
          env:
            - name: PUID
              value: "0"
            - name: PGID
              value: "0"
            - name: UMASK
              value: "022"
      volumes:
        - name: alist-data-volume
          persistentVolumeClaim:
            claimName: alist-data
```





## 创建PVC 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alist-data
  namespace: wp-cluster
spec:
  storageClassName: nfs-wp-cluster
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1024Gi
root@k8s-made-01-32:/yaml/alist#
root@k8s-made-01-32:/yaml/alist#
root@k8s-made-01-32:/yaml/alist# cat alist-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alist-data
  namespace: wp-cluster
spec:
  storageClassName: nfs-wp-cluster
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1024Gi
```





## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: alist-service
spec:
  type: NodePort
  selector:
    app: alist
  ports:
    - protocol: TCP
      port: 5244
      nodePort: 30099
```





## 部署过程

```bash
## 部署PVC 
kubectl apply -f /yaml/alist/alist-pvc.yaml -n wp-cluster

## 部署Deployment
kubectl apply -f /yaml/alist/alist-deploy.yaml -n wp-cluster

## 部署Service
kubectl apply -f /yaml/alist/alist-svc.yaml -n wp-cluster

## 查看服务运行情况
root@k8s-made-01-32:~# kubectl get pod -n wp-cluster
NAME                                                              READY   STATUS    RESTARTS        AGE
alist-deployment-7fcc4596bf-plv2k                                 1/1     Running   0               8s
busybox-pod                                                       1/1     Running   144 (55m ago)   6d
nfs-wp-cluster-nfs-subdir-external-provisioner-77d6fc75dc-cddd8   1/1     Running   0               6d5h
wordpress-577d59bc58-7nn9w                                        1/1     Running   0               107m
wordpress-577d59bc58-gv595                                        1/1     Running   0               107m
wordpress-577d59bc58-xx2pz                                        1/1     Running   0               107m
wpdb57-85bfc9cd76-t4kvz                                           1/1     Running   0               6d5h

## 获取管理员密码
root@k8s-made-01-32:~# kubectl logs -n wp-cluster alist-deployment-7fcc4596bf-plv2k
INFO[2023-11-20 07:25:22] reading config file: data/config.json
INFO[2023-11-20 07:25:22] config file not exists, creating default config file
INFO[2023-11-20 07:25:22] load config from env with prefix:
INFO[2023-11-20 07:25:22] init logrus...
INFO[2023-11-20 07:25:22] Successfully created the admin user and the initial password is: joeEcoq5
INFO[2023-11-20 07:25:23] start HTTP server @ 0.0.0.0:5244
```





## 添加存储

挂载路径就是前端显示的目录名字

根文件路径就是实际服务器后端的存储路径

![image-20231120170714783](https://cdn1.ryanxin.live/image-20231120170714783.png)



## 密码找回

3.25.0以上版本将密码改成加密方式存储的hash值，无法直接反算出密码，如果忘记了密码只能通过重新 **`随机生成`** 或者 **`手动设置`**

```bash
# 随机生成一个密码
docker exec -it alist ./alist admin random
# 手动设置一个密码,`NEW_PASSWORD`是指你需要设置的密码
docker exec -it alist ./alist admin set NEW_PASSWORD
```



