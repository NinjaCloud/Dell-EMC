=============================================================
# Lab : StatefulSet Implementation
=============================================================

----------------------------------------------------------------------
## Task 1: Create Stateful Set
----------------------------------------------------------------------

### Download the YAML definition for an Nginx StatefulSet
```sh
wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/nginx-sts.yaml
```

### Create a StatefulSet by applying the YAML
```sh
kubectl apply -f nginx-sts.yaml
```

### Verify the headless service
```sh
kubectl get service nginx-svc
```

### Validate the StatefulSet creation
```sh
kubectl get statefulset nginx-sts
```

### Delete all pods
```sh
kubectl delete pod --all
```

### Verify pod status
```sh
kubectl get pod
```

### Delete a specific pod
```sh
kubectl delete pod web-0
```

### Verify pod recreation
```sh
kubectl get pod
```

----------------------------------------------------------------------------
## Task 2: Scaling a StatefulSet
----------------------------------------------------------------------------

### Scale the StatefulSet to 5 replicas
```sh
kubectl scale sts nginx-sts --replicas=5
```

### Verify the pods getting created in ordinal order
```sh
kubectl get pods -w -l app=nginx-sts
```

### Verify the PersistentVolumeClaims getting created in ordinal order
```sh
kubectl get pvc -l app=nginx-sts
```

### Edit the StatefulSet YAML to reduce replicas to 3
```sh
kubectl edit sts nginx-sts
```

### Observe that the controller deletes the pods one at a time
```sh
kubectl get pods -w -l app=nginx-sts
```

### Verify StatefulSet’s PersistentVolumeClaims are not deleted on scaling down
```sh
kubectl get pvc -l app=nginx-sts
```

-------------------------------------------------------------------------------
## Task 3: Delete a StatefulSet - Cleanup
--------------------------------------------------------------------------------

### Delete the StatefulSet
```sh
kubectl delete -f nginx-sts.yaml
```

### List all PersistentVolumeClaims allocated to StatefulSet pods
```sh
kubectl get pvc
```

### Delete all PersistentVolumeClaims
```sh
kubectl delete pvc --all
```

----------------------------------------------------------------------------
## YAML Definition: nginx-sts.yaml
----------------------------------------------------------------------------
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-sts
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

----------------------------------------------------------------------------
## YAML Definition: Headless Service (nginx-svc.yaml)
----------------------------------------------------------------------------
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  clusterIP: None
  selector:
    app: nginx-sts
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
