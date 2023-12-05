# kubernetes-volumes

## Volumes
To persist data of applications. Lets say we have a SQL database and we created a pod for it. If pod gets deleted, all of our data is lost.
To tackle this problem, we should have more robust and consistant solutions. Volumes comes to play here.
Volumes can be created to persist data from any storage type (local storage, aws s3 or network file system).

These are also used if we need to mount a local directory to container directory to store/retrieve config files or log files.

Note: If we rae not using any robust and consistant storage solutions (aws s3), then we have to configure the storage to make it consistant and manage disaster recovery.
i.e, if we are using local storage, then we have to configure local storages on all the nodes, so that if pod dies on one node and created on another node the storage is accessible. 

### Persistant Volume
A kubernetes resource. 

### Persistent Volume Claims
An abstraction over persistent volumes. Whenever, PVC is created, it matches the requirement with available PV. If it does not find any PV according to requirements, it dynamically proviison the PV.

### Manifest files for Volumes
1- Create a pvc for application (databases) like in mysql-pvc.yaml

2- Configure this pvc into spec of pod under volumes attribute, like this;
```
spec:
      volumes: 
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

3- Mount this volume to container's directory like this;
```
spec:
      volumes: 
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
      containers:
        - name: sql-container
          image: mysql:latest
          ports:
            - containerPort: 3306
          volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /var/lib/mysql # we have to check which directory in container particular db stores data, eg, for mongodb, its data/db.
```

To get events by timestamp;
```
kubectl get events --sort-by='.metadata.creationTimestamp'
```

After volume is successfully mounted, if we add any data in pods condifgured directory, it will be persistent even if pod dies.

### How to mount a local directory to container

1- We create a pv and mention the directory path you want to mount in `hostPath`;
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/home/apmec/kubernetes-volumes"
```

2- We create a pvc (pv and pvc should be of same storage capacity. Also, pvc is namespaced but pv are not);
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/home/apmec/kubernetes-volumes"
```

3- Mount the pv using pvc to container directory in its spec. See `mongodb.yaml` for example.
