=============================================================
# Lab : Deployment
=============================================================

----------------------------------------------------------------------
## Task 1: Write a Deployment yaml and Apply it
----------------------------------------------------------------------

### Create a `dep-nginx.yaml` using the content given below:
```sh
vi dep-nginx.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx-dep
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.12.2
        ports:
        - containerPort: 80
```

### Apply the Deployment yaml created in the previous step:
```sh
kubectl apply -f dep-nginx.yaml
```

### View the objects created by Kubernetes, Deployment, and Replica Set:
```sh
kubectl get deployments
kubectl get rs
```

### Access one of the Pods and view nginx version:
```sh
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash
```
```sh
nginx -v
exit
```

-------------------------------------------------------------------------
## Task 2: Update the Deployment with a Newer Image
-------------------------------------------------------------------------

### Update the nginx image in Pod:
```sh
kubectl set image deployment/nginx-dep nginx-ctr=nginx:1.11
```

### Describe the deployment and see that the old pods are replaced with newer ones:
```sh
kubectl describe deployments
```

### Access one of the Pods and view nginx version:
```sh
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash
```
```sh
nginx -v
exit
```

-----------------------------------------------------------------------------
## Task 3: Rollback of Deployment
-----------------------------------------------------------------------------

### View the history of Deployments:
```sh
kubectl rollout history deployment/nginx-dep
```

### Rollback the Deployment done in the previous task:
```sh
kubectl rollout undo deployment/nginx-dep --to-revision=1
```
```sh
kubectl get rs
```

### Access one of the Pods and view nginx version:
```sh
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash
```
```sh
nginx -v
exit
```

------------------------------------------------------------------------------
## Task 4: Scaling of Deployments
------------------------------------------------------------------------------

### View the number of Pod replicas created by the Deployment:
```sh
kubectl get deployments
kubectl get pods
```

### Scale up the deployment to have 8 Pod replicas:
```sh
kubectl scale deployment nginx-dep --replicas=8
```

### Check the Pods and deployment to verify that the number of Pod replicas are 8:
```sh
kubectl get deployments
kubectl get pods
```

### Scale down the deployments to 2 Pod replicas:
```sh
kubectl scale deployment nginx-dep --replicas=2
```

### Check the Pods and deployment to verify that the number of Pod replicas are down to 2:
```sh
kubectl get deployments
kubectl get pods
```

-----------------------------------------------------------------------------
## Task 6: Cleanup the resources using the command below
-----------------------------------------------------------------------------
```sh
kubectl delete -f dep-nginx.yaml
