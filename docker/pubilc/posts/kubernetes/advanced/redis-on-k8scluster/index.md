# k8s中部署单节点redis

k8s中部署单节点redis

https://gitee.com/zdevops/k8s-yaml/blob/main/redis/single/





## **redis-cm.yaml**

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
  namespace: zdevops
data:
  redis-config: |
    appendonly yes
    protected-mode no
    dir /data
    port 6379
    requirepass redis@abc.com

```





## **redis-sts.yaml**

```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: zdevops
  labels:
    app: redis
spec:
  serviceName: redis-headless
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: 'registry.zdevops.com.cn/library/redis:6.2.7'
          command:
            - "redis-server"
          args:
            - "/etc/redis/redis.conf"
          ports:
            - name: redis-6379
              containerPort: 6379
              protocol: TCP
          volumeMounts:
            - name: config
              mountPath: /etc/redis
            - name: data
              mountPath: /data
          resources:
            limits:
              cpu: '2'
              memory: 4000Mi
            requests:
              cpu: 100m
              memory: 500Mi
      volumes:
        - name: config
          configMap:
            name: redis-config
            items:
              - key: redis-config
                path: redis.conf
  volumeClaimTemplates:
    - metadata:
        name: data
        namespace: zdevops
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "glusterfs"
        resources:
          requests:
            storage: 5Gi

---
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
  namespace: zdevops
  labels:
    app: redis
spec:
  ports:
    - name: redis-6379
      protocol: TCP
      port: 6379
      targetPort: 6379
  selector:
    app: redis
  clusterIP: None
  type: ClusterIP

```



---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/kubernetes/advanced/redis-on-k8scluster/  

