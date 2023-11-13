
## namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: wp-cluster
```
<br>

## mysql deployment
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wpdb57
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wpdb57
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wpdb57
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
      containers:
        - name: wpdb57
          image: mysql:5.7
          env:
            - name: TZ
              value: Asia/Shanghai
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: wpdb-secret
                  key: password
            - name: MYSQL_DATABASE
              value: wpdb
            - name: MYSQL_USER
              value: wp_user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: wpdb-secret
                  key: password
          ports:
            - name: mysql
              containerPort: 3306
          volumeMounts:
            - name: mysql-data-volume
              mountPath: /var/lib/mysql
            - name: mysql-log-volume
              mountPath: /var/log/mysql
            - name: wpdb57-config
              mountPath: /etc/mysql/conf.d/custom.cnf
              subPath: custom.cnf
              readOnly: true
          resources:
            requests:
              memory: "2Gi"
              cpu: "1"
            limits:
              memory: "8Gi"
              cpu: "2"
      volumes:
        - name: mysql-data-volume
          persistentVolumeClaim:
            claimName: wpdb-data-pvc
        - name: mysql-log-volume
          persistentVolumeClaim:
            claimName: wpdb-log-pvc
        - name: wpdb57-config
          configMap:
            name: wpdb57-config
            items:
              - key: custom.cnf
                path: custom.cnf

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: wpdb57-config
data:
  custom.cnf: |
    [mysqld]
    default_authentication_plugin=mysql_native_password
    skip-name-resolve
    datadir=/var/lib/mysql
    bind-address=0.0.0.0

    log-error=/var/log/mysql/error.log
    slow_query_log=1
    long_query_time=3
    slow_query_log_file=/var/log/mysql/slow_query.log

    # replication
    log-bin=binlog
    binlog_format=ROW
    server-id=1
    innodb_flush_log_at_trx_commit=1
    sync_binlog=1

    # fulltext index
    ngram_token_size=1

    # required by confluence
    default_storage_engine=InnoDB
    character-set-server=utf8mb4
    collation-server=utf8mb4_bin
    max_allowed_packet=256M
    innodb_log_file_size=2GB
    transaction-isolation=READ-COMMITTED
    binlog_format=row

    # required by quartz
    # will make table name case-insensitive
    lower_case_table_names=1

    # sql mode disable full_group for backward compatibility
    sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

---
apiVersion: v1
kind: Service
metadata:
  name: mpdb57-nodeport
spec:
  type: NodePort
  ports:
    - name: mysql
      port: 3306
      targetPort: mysql
      nodePort: 30357
      protocol: TCP
  selector:
    app: mpdb57


---
apiVersion: v1
kind: Service
metadata:
  name: mpdb57
spec:
  type: ClusterIP
  ports:
    - name: mysql
      port: 3306
      targetPort: mysql
      protocol: TCP
  selector:
    app: mpdb57


```

```bash
kubectl create secret generic wpdb-secret --from-literal=password=YourMySQLPassword
```


```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wpdb-data-pvc
  namespace: wp-cluster
spec:
  storageClassName: nfs-wp-cluster
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 150Gi

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wpdb-log-pvc
  namespace: wp-cluster
spec:
  storageClassName: nfs-wp-cluster
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

<br>

## WordPress deployment

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wordpress
  name: wordpress
spec:
  selector:
    app: wordpress
  type: NodePort
  ports:
  - protocol: TCP
    port: 8099
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wp-cluster
  labels:
    app: wordpress
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - image: wordpress
        name: wordpress
        env:
        - name: WORDPRESS_DB_NAME
          value: wpdb
        - name: WORDPRESS_DB_USER
          value: wpuser
        - name: WORDPRESS_DB_PASSWORD
          value: Ceamg.com
        - name: WORDPRESS_DB_HOST
          value: wpdb.default.svc.cluster.local
        volumeMounts:
        - mountPath: "/var/www/html"
          name: wp-data-volume     
      volumes:
        - name: wp-data-volume
          persistentVolumeClaim:
            claimName: wp-data-pvc
```


```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-data-pvc
  namespace: wp-cluster
spec:
  storageClassName: nfs-wp-cluster
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
```

<br>

## NFS Provisioner Values

```yaml
nfs:
  server: 10.1.0.38
  path: /data/k8s/wp-cluster
  mountOptions:
    - vers=4
#    - minorversion=0
#    - rsize=1048576
#    - wsize=1048576
#    - hard
#    - timeo=600
#    - retrans=2
#    - noresvport

storageClass:
  name: nfs-wp-cluster
  defaultClass: false
  allowVolumeExpansion: true
  reclaimPolicy: Delete
  provisionerName: nfs-wp-cluster
  archiveOnDelete: true
```

```bash
#添加harbor私有仓库
helm repo add --username=xceo --password=Ceamg.com1234 ceamg https://harbor.ceamg.com/chartrepo/chart \
--ca-file /etc/containerd/certs.d/harbor.ceamg.com/ca.crt \
--cert-file  /etc/containerd/certs.d/harbor.ceamg.com/harbor.ceamg.com.crt \
--key-file /etc/containerd/certs.d/harbor.ceamg.com/harbor.ceamg.com.key

"ceamg" has been added to your repositories
```

```bash
root@k8s-made-01-32:/yaml/nfs# helm repo list
NAME                            URL
nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
rmxc                            https://harbor.rmxc.tech/chartrepo/charts
ceamg                           https://harbor.ceamg.com/chartrepo/chart
bitnami                         https://charts.bitnami.com/bitnami
jetstack                        https://charts.jetstack.io
```


```bash
helm install -n wp-cluster -f /yaml/nfs/nfs-provisioner.value.yaml nfs-wp-cluster ceamg/nfs-subdir-external-provisioner
```

<br>

## helm install
```bash
root@k8s-made-01-32:/yaml/nfs# helm install -n wp-cluster -f /yaml/nfs/nfs-provisioner.value.yaml nfs-wp-cluster ceamg/nfs-subdir-external-provisioner
NAME: nfs-wp-cluster
LAST DEPLOYED: Mon Nov 13 15:46:52 2023
NAMESPACE: wp-cluster
STATUS: deployed
REVISION: 1
TEST SUITE: None
```


```bash
root@k8s-made-01-32:/yaml/wp-cluster# kubectl get pod -n wp-cluster
NAME                                                              READY   STATUS    RESTARTS   AGE
nfs-wp-cluster-nfs-subdir-external-provisioner-77d6fc75dc-hfxrb   1/1     Running   0          51m
```

<br>



## 创建wpdb-pvc

```bash
root@k8s-made-01-32:/yaml/wp-cluster# kubectl apply -f wpdb-pvc.yaml
persistentvolumeclaim/wpdb-data-pvc created
persistentvolumeclaim/wpdb-log-pvc created

```


<br>

```bash
root@k8s-made-01-32:/yaml/wp-cluster# kubectl get pvc -n wp-cluster
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
wpdb-data-pvc   Bound    pvc-92a0cbb3-a060-4d5c-9c87-66f87ad802e9   150Gi      RWO            nfs-wp-cluster   19s
wpdb-log-pvc    Bound    pvc-b0811659-797d-4e91-9cb9-6c87ab8a2812   50Gi       RWO            nfs-wp-cluster   19s

```

<br>


## 创建wpdata-pvc

```bash
root@k8s-made-01-32:/yaml/wp-cluster# kubectl apply -f wp-data.yaml
persistentvolumeclaim/wp-data-pvc created
```

```bash
root@k8s-made-01-32:/yaml/wp-cluster# kubectl get pvc -n wp-cluster
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
wp-data-pvc     Bound    pvc-e3e1b943-b567-49e6-a7bf-ed8f0534ab2c   200Gi      RWO            nfs-wp-cluster   3s
wpdb-data-pvc   Bound    pvc-92a0cbb3-a060-4d5c-9c87-66f87ad802e9   150Gi      RWO            nfs-wp-cluster   2m12s
wpdb-log-pvc    Bound    pvc-b0811659-797d-4e91-9cb9-6c87ab8a2812   50Gi       RWO            nfs-wp-cluster   2m12s
```



## 创建mysql-secret

```bash

root@k8s-made-01-32:/yaml/wp-cluster# kubectl create secret generic -n wp-cluster  wpdb-secret --from-literal=password=Ceamg.com
secret/wpdb-secret created

root@k8s-made-01-32:/yaml/wp-cluster# kubectl get secrets -n wp-cluster
NAME                                   TYPE                 DATA   AGE
sh.helm.release.v1.nfs-wp-cluster.v1   helm.sh/release.v1   1      62m
wpdb-secret                            Opaque               1      16s
```


## 创建mysql-pod

```bash
root@k8s-made-01-32:/yaml/wp-cluster# kubectl apply -f wpdb-deploy.yaml
service/wpdb created
deployment.apps/wpdb created
```


## 创建wordpress Pod

```bash

```




```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql57
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql57
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql57
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
      containers:
        - name: mysql57
          image: mysql:5.7
          env:
            - name: TZ
              value: Asia/Shanghai
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql57-secrets
                  key: root
            - name: MYSQL_DATABASE
              value: test
            - name: MYSQL_USER
              value: test_user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql57-secrets
                  key: test_user
          ports:
            - name: mysql
              containerPort: 3306
          volumeMounts:
            - name: mysql57-data
              mountPath: /var/lib/mysql
            - name: mysql57-log
              mountPath: /var/log/mysql
            - name: mysql57-config
              mountPath: /etc/mysql/conf.d/custom.cnf
              subPath: custom.cnf
              readOnly: true
          resources:
            requests:
              memory: "2Gi"
              cpu: "1"
            limits:
              memory: "4Gi"
              cpu: "2"
      volumes:
        - name: mysql57-data
          persistentVolumeClaim:
            claimName: mysql57-data
        - name: mysql57-log
          persistentVolumeClaim:
            claimName: mysql57-log
        - name: mysql57-config
          configMap:
            name: mysql57-config
            items:
              - key: custom.cnf
                path: custom.cnf

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql57-config
data:
  custom.cnf: |
    [mysqld]
    default_authentication_plugin=mysql_native_password
    skip-name-resolve
    datadir=/var/lib/mysql
    bind-address=0.0.0.0

    log-error=/var/log/mysql/error.log
    slow_query_log=1
    long_query_time=3
    slow_query_log_file=/var/log/mysql/slow_query.log

    # replication
    log-bin=binlog
    binlog_format=ROW
    server-id=1
    innodb_flush_log_at_trx_commit=1
    sync_binlog=1

    # fulltext index
    ngram_token_size=1

    # required by confluence
    default_storage_engine=InnoDB
    character-set-server=utf8mb4
    collation-server=utf8mb4_bin
    max_allowed_packet=256M
    innodb_log_file_size=2GB
    transaction-isolation=READ-COMMITTED
    binlog_format=row

    # required by quartz
    # will make table name case-insensitive
    lower_case_table_names=1

    # sql mode disable full_group for backward compatibility
    sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

---
apiVersion: v1
kind: Service
metadata:
  name: mysql57-nodeport
spec:
  type: NodePort
  ports:
    - name: mysql
      port: 3306
      targetPort: mysql
      nodePort: 30357
      protocol: TCP
  selector:
    app: mysql57


---
apiVersion: v1
kind: Service
metadata:
  name: mysql57
spec:
  type: ClusterIP
  ports:
    - name: mysql
      port: 3306
      targetPort: mysql
      protocol: TCP
  selector:
    app: mysql57

```


```bash
exit;

kubectl create secret generic mysql57-secrets \
  --from-literal=root="$(openssl rand -hex 12)" \
  --from-literal=test_user="$(openssl rand -hex 12)"
```


https://blog.csdn.net/zwqjoy/article/details/112243757