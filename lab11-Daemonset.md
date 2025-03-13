=============================================================
# Lab : DaemonSet in Kubernetes
=============================================================

----------------------------------------------------------------------
## Create a DaemonSet using the YAML file
----------------------------------------------------------------------

### Create a `ds-pod.yaml` file:
```sh
vi ds-pod.yaml
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-ds
  labels:
    app: fluent-ds
spec:
  selector:
    matchLabels:
      app: fluentd-app
  template:
    metadata:
       labels:
           app: fluentd-app
    spec:
      containers:
      - name: fluentd-ctr
        image: fluent/fluentd:v1.16
```

### Apply the YAML definition to create a `fluent-ds` DaemonSet:
```sh
kubectl apply -f ds-pod.yaml
```

### Check the available DaemonSets in the Kubernetes cluster:
```sh
kubectl get ds fluent-ds
```

### Verify that pods for Fluentd are created—one for each node using DaemonSet:
```sh
kubectl get pods -o wide
```

----------------------------------------------------------------------
## Cleanup the DaemonSet
----------------------------------------------------------------------

### Delete the DaemonSet using the command below:
```sh
kubectl delete -f ds-pod.yaml
```

### Verify the pods to confirm that all Fluentd pods have been deleted from each node:
```sh
kubectl get pods
