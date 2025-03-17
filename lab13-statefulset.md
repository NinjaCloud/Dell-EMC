=============================================================
# Lab : StatefulSet Implementation
=============================================================

----------------------------------------------------------------------
## Task 1: Create Stateful Set
----------------------------------------------------------------------

### Download file with YAML definition for an nginx StatefulSet
```sh
wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/nginx-sts.yaml
```

### Create a StatefulSet by applying the YAML
```sh
kubectl apply -f nginx-sts.yaml
```

### Create a headless service
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

### Edit the StatefulSet YAML and reduce replicas to 3
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

### List all the PV and PVCs allocated to StatefulSet pods
```sh
kubectl get pvc
```

### Delete all PersistentVolumeClaims
```sh
kubectl delete pvc --all
