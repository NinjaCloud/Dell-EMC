# Lab: Services in Kubernetes

## Task 1: Create a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    env: prod 
    type: front-end
    app: httpd-ws
spec:
  containers:
  - name: httpd-container
    image: httpd
    ports:
       - containerPort: 80
```

### Apply the Pod definition
```sh
kubectl create -f httpd-pod.yaml
```

### Verify Pod Creation
```sh
kubectl get pods
kubectl get pods -o wide
kubectl describe pod httpd-pod
```

---

## Task 2: Setup ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### Apply the ClusterIP service
```sh
kubectl apply -f httpd-svc.yaml
```

### Verify Service Details
```sh
kubectl get svc
kubectl describe svc httpd-svc
kubectl get ep  
```

### Check External IPs of Nodes
```sh
kubectl get nodes -o wide | awk '{print $7}'
ssh -t ubuntu@<Node_IP> curl <Cluster_IP>:<Service_Port>
```

---

## Task 3: Setup NodePort Service

Modify `httpd-svc.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

### Apply the Changes
```sh
kubectl apply -f httpd-svc.yaml
```

### Verify Service Details
```sh
kubectl describe svc httpd-svc
curl <EXTERNAL-IP>:NodePort
```

### Check External IPs of Nodes
```sh
kubectl get nodes -o wide | awk '{print $7}'
ssh -t ubuntu@<Node_IP> curl <Cluster_IP>:<Service_Port>
```

---

## Task 4: Setup LoadBalancer Service

Modify `httpd-svc.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### Apply the Changes
```sh
kubectl apply -f httpd-svc.yaml
```

### Verify Service Details
```sh
kubectl get svc
kubectl describe svc httpd-svc
curl <LoadBalancer_DNS>
```

---

## Task 5: Delete and Recreate httpd Pod

### Delete Existing Pod
```sh
kubectl delete -f httpd-pod.yaml
```

### Check Empty Endpoints
```sh
kubectl describe svc httpd-svc
```

### Recreate the Pod
```sh
kubectl apply -f httpd-pod.yaml
kubectl describe svc httpd-svc
```

---

## Task 6: Cleanup Resources
```sh
kubectl delete -f httpd-pod.yaml
kubectl delete -f httpd-svc.yaml

