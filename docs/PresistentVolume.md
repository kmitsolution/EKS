### create namespace
To separate your cluster resources logically it is *best practice* to use _namespaces_. You can separate either by project, customer, team, environment,...
The benefit you gain is by getting control over resource qoutas, access control etc

```
    kubectl create namespace ns-eks
```
### Create a storageclass (gp2-storage.yaml)

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
mountOptions:
  - debug
```

```
# Run below command to create above storage class

kubectl apply -f gp2-storage-class.yaml --namespace=ns-eks
```
>   check current situation:
>    ```
>    kubectl get storageclasses --namespace=ns-eks
>    ```
>    set default:
>    ```
>    kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}' --namespace=ns-eks
>    ```
>    check again:
>    ```
>    kubectl get storageclasses --namespace=ns-eks
>    ```


### define a persistent volume claim (pvcs.yaml)

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```

```
kubectl apply -f pvcs.yaml --namespace=ns-eks
```
and check:
```
kubectl get pvc --namespace=ns-eks
```

# deploy mysql
## create secret which stores mysql pw, to be injected as env var into container
```
kubectl create secret generic mysql-pass --from-literal=password=eks-course-mysql-pw --namespace=ns-eks
```
check:
```
kubectl get secrets --namespace=ns-eks

```

## create deploy-mysql.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim

```
## launch mysql deployment
```
kubectl apply -f deploy-mysql.yaml --namespace=ns-eks
```
## checks
* persistent volumes
* persistent volume claims
* pods
```
kubectl get pv --namespace=ns-eks
kubectl get pvc --namespace=ns-eks
kubectl get pods -o wide --namespace=ns-eks
```
* EBS volumes
goto AWS mgm console => EC2 => Elastic Block store => volumes


# deploy wordpress (deploy-wordpress.yaml)

```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim

```

```
kubectl apply -f deploy-wordpress.yaml --namespace=ns-eks
```
get URL of the app:
```
kubectl describe service wordpress --namespace=ns-eks | grep Ingress
```
or goto AWS console => EC2

### Create stateful 
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress-statefulset
  labels:
    app: wordpress-statefulset
spec:
  ports:
    - port: 80
  selector:
    app: wordpress-statefulset
    tier: frontend
  type: LoadBalancer
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: StatefulSet
metadata:
  name: wordpress-statefulset
  labels:
    app: wordpress-statefulset
spec:
  selector:
    matchLabels:
      app: wordpress-statefulset
      tier: frontend
  replicas: 1
  serviceName: wordpress-statefulset-frontend
  template:
    metadata:
      labels:
        app: wordpress-statefulset
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
        ports:
        - containerPort: 80
          name: wordpress
  volumeClaimTemplates:
  - metadata:
      name: wordpress-persistent-storage
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
      storageClassName: gp2

```
# cleanup
delete Wordpress _deployment_
```
kubectl delete -f deploy-wordpress-by-deployment.yaml --namespace=ns-eks
kubectl delete -f deploy-wordpress-by-statefulset.yaml --namespace=ns-eks
```

delete MySQL _deployment_
```
kubectl delete -f deploy-mysql.yaml --namespace=ns-eks
```

delete EBS volumes  
since we specified policy _retain_ for the pv's, we have to delete them manually after deleting the pods, which had them in use.
For that go to AWS mgm console => EC2 => Volumes => select and delete the detached ones
