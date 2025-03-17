 =============================================================
# Lab : Persistent Volume in Kubernetes
=============================================================

----------------------------------------------------------------------------
## Task 1: Get Node Label and Create Custom Index.html on Node
----------------------------------------------------------------------------

### View worker nodes and their labels
```sh
kubectl get nodes --show-labels | grep kubernetes.io/hostname
```

### Get the public IP of a node
```sh
kubectl get nodes -o wide
```

### SSH to the selected node
```sh
ssh -t ubuntu@<node_public_IP>
```

### Switch to root and create a directory with a custom index.html
```sh
sudo su
mkdir /pvdir
echo "Hello World!" > /pvdir/index.html
```

--------------------------------------------------------------------------------
## Task 2: Create a Local Persistent Volume
--------------------------------------------------------------------------------

### Create a PersistentVolume YAML definition
```sh
vi pv-volume.yaml
```
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pvdir"
```

### Apply the PersistentVolume YAML
```sh
kubectl apply -f pv-volume.yaml
```

### Verify PersistentVolume creation
```sh
kubectl get pv
kubectl describe pv pv-volume
```

------------------------------------------------------------------------------------
## Task 3: Create a PersistentVolumeClaim
------------------------------------------------------------------------------------

### Create a PersistentVolumeClaim YAML definition
```sh
vi pv-claim.yaml
```
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

### Apply the PersistentVolumeClaim YAML
```sh
kubectl apply -f pv-claim.yaml
```

----------------------------------------------------------------------------------------
## Task 4: Create an Nginx Pod with NodeSelector
----------------------------------------------------------------------------------------

### Create a Pod YAML definition
```sh
vi pv-pod.yaml
```
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
    - name: pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
  nodeSelector:
    kubernetes.io/hostname: <node_name>
```

### Apply the Pod YAML
```sh
kubectl apply -f pv-pod.yaml
```

### Verify Pod creation and node placement
```sh
kubectl get pods -o wide
```

### Access the container shell
```sh
kubectl exec -it pv-pod -- /bin/bash
```

### Verify PersistentVolume inside the container
```sh
apt-get update
apt-get install curl -y
curl localhost
exit
```

### Cleanup: Delete resources created in this lab
```sh
kubectl delete -f pv-pod.yaml
kubectl delete -f pv-claim.yaml
kubectl delete -f pv-volume.yaml
