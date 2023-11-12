# 


## mysql deployment
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wpdb
  labels:
    app: wpdb
spec:
  type: ClusterIP
  selector:
    app: wpdb
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
---    
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wpdb
  labels:
    app: wpdb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wpdb
  template:
    metadata:
      labels:
        app: wpdb
    spec:
      containers:
      - image: mysql:5.7.36
        imagePullPolicy: IfNotPresent
        name: wpdb
        env:
        - name: MYSQL_DATABASE
          value: wpdb
        - name: MYSQL_USER
          value: wpuser
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: wpdb-secret
              key: password
        volumeMounts:
        - mountPath: "/var/lib/mysql"
          name: mysql-data-volume
        - mountPath: "/var/log/mysql"
          name: mysql-log-volume        
      volumes:
        - name: mysql-data-volume
          persistentVolumeClaim:
            claimName: wpdb-data-pvc
        - name: mysql-log-volume
          persistentVolumeClaim:
            claimName: wpdb-log-pvc


```

```bash
kubectl create secret generic wpdb-secret --from-literal=password=YourMySQLPassword
```



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

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/kubernetes/advanced/wordpress-mysql-cluster-nfs/  

