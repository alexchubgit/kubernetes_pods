## Kubernetes Pods

### Ingress controller


## Pods

<!-- [Deploy A Sample Nginx Application](README/pods.md) -->

**Now that we have all the components to make the cluster and applications work, let’s deploy a sample Nginx application and see if we can access it over a NodePort**

**Create an Nginx deployment. Execute the following directly on the command line. It deploys the pod in the default namespace.**


### Nginx

```yaml
cat <<EOF | kubectl apply -f -

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
EOF
```

```yaml
cat <<EOF | kubectl apply -f -

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
EOF
```


### MySQL

```yaml
cat <<EOF | kubectl apply -f -

apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: kubernetes.io/basic-auth
stringData:
  password: test1234
EOF
```

```yaml
cat <<EOF | kubectl apply -f -

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
EOF
```

```yaml
cat <<EOF | kubectl apply -f -

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: password
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
EOF
```

```yaml
cat <<EOF | kubectl apply -f -

apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    run: mysql
spec:
  selector:
    app: mysql
  type: NodePort
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30036
      protocol: TCP
EOF
```
