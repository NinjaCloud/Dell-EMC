## Cluster Setup using Kops

### Task-1:  Launching EC2 instances and Connecting to EC2 Instances using SSH

* Manually Launch a `t2.micro` instance with OS version as `Ubuntu 22.04 LTS` in North Virginia (us-east-1) Region.
* Enable `SSH`, `HTTP`, `HTTPS` and `edit`.
* Type : `Custom TCP`     Source Type: `Anywhere`    Port Range : `8080-8090`
* Configure Storage: `10 GiB`
* Once Launched, Connect to the Instance using `MobaXterm` or `Putty` or `EC2 Instance Connect` with username "`ubuntu`".

### Task 2: Create an IAM role

* Create an IAM role named "kops-admin-role" with the "AdministratorAccess" policy attached.
* Now, associate the IAM role "kops-admin-role" with your EC2 instance named "kops" by following these steps:
  ** Navigate to the EC2 console and select the "kops" instance.
  ** Click on "Action" > "Security" > "Modify IAM role."
  ** Search for the "kops-admin-role" role, select it, and click "Update IAM role."

### Task 3: Setting up a Kubernetes Cluster
Set the hostname to "kops"
```
sudo hostnamectl set-hostname kops
```
Open a new bash shell
```
bash
```
```
vi kops.sh
```
```
#!/bin/bash

echo "Let's get started with Kubernetes cluster creation using KOPS!"
echo "Enter your name:"
read username
lower_username=$(echo -e $username | sed 's/ //g' | tr '[:upper:]' '[:lower:]')
date_now=$(date "+%F-%H-%m")
clname=$(echo $lower_username-$date_now.k8s.local)
echo "Your Kubernetes cluster name will be $clname"

TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

az=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone)
region=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/region)

sudo sed -i "/$nrconf{restart}/d" /etc/needrestart/needrestart.conf
echo "\$nrconf{restart} = 'a';" | sudo tee -a /etc/needrestart/needrestart.conf
export DEBIAN_FRONTEND=noninteractive
export NEEDRESTART_MODE=a

sudo apt update -y
sudo apt install nano curl python3-pip -y
sudo snap install aws-cli --classic

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

# Install kops
curl -LO "https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64"
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

# Generate SSH key
ssh-keygen -t rsa -N "" -f $HOME/.ssh/id_rsa

# Create S3 bucket for kops state store
aws s3 mb s3://$clname --region $region

# Set KOPS_STATE_STORE environment variable
export KOPS_STATE_STORE=s3://$clname

# Create Kubernetes cluster
kops create cluster --node-count=2 --master-size="t2.medium" --node-size="t2.medium" --master-volume-size=20 --node-volume-size=20 --zones $az --name $clname --ssh-public-key ~/.ssh/id_rsa.pub --yes
kops update cluster $clname --yes

# Export KOPS_STATE_STORE to bashrc
echo "export KOPS_STATE_STORE=s3://$clname" >> /home/ubuntu/.bashrc
source /home/ubuntu/.bashrc

# Export kubectl configuration
kops export kubecfg --admin

# Validate cluster
for (( x=0 ; x < 30 ; x++ )); do
  echo "Validating Cluster"
  if kops validate cluster > status.txt 2>/dev/null && grep -q "is ready" status.txt; then
    echo "Your Cluster is now ready!"
    break
  else
    sleep 20
    echo "x: $x"
  fi
done

# Create Kubernetes Dashboard
cat > kubernetes-dashboard.yaml <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard

---

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 32000
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kubernetes-dashboard
type: Opaque

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-csrf
  namespace: kubernetes-dashboard
type: Opaque
data:
  csrf: ""

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-key-holder
  namespace: kubernetes-dashboard
type: Opaque

---

kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs", "kubernetes-dashboard-csrf"]
    verbs: ["get", "update", "delete"]
    # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["kubernetes-dashboard-settings"]
    verbs: ["get", "update"]
    # Allow Dashboard to get metrics.
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["heapster", "dashboard-metrics-scraper"]
    verbs: ["proxy"]
  - apiGroups: [""]
    resources: ["services/proxy"]
    resourceNames: ["heapster", "http:heapster:", "https:heapster:", "dashboard-metrics-scraper", "http:dashboard-metrics-scraper"]
    verbs: ["get"]

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
rules:
  # Allow Metrics Scraper to get metrics from the Metrics server
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.4.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
              # Create on-disk volume to store exec logs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
      securityContext:
        fsGroup: 2001
      serviceAccountName: kubernetes-dashboard
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
---

kind: ServiceAccount
apiVersion: v1
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
EOF

kubectl apply -f kubernetes-dashboard.yaml

# Retrieve URL of the Kubernetes Dashboard
urls=()
for i in {2..4}; do
  urls+=("https://$(kubectl get nodes -o wide -n kubernetes-dashboard | awk 'NR=='$i'{print $7}')":32000)
done

# Generate token.txt file with URLs and token
{
  echo "                                                   "
  echo "********        HERE ARE THE DETAILS REQUIRED        ********"
  echo "******** You can use any one of the below given URLs ********"
  for url in "${urls[@]}"; do
    echo "URL: $url"
  done
  echo "Creating Token"
  echo "Token is:"
  kubectl -n kube-system describe secret "$(kubectl -n kube-system get secret | grep kops-admin | awk '{print $1}')" | grep token: | awk '{print $2}'
  echo "******************          END          ******************"
  echo "                                                   "
} > token.txt

cat token.txt

```
Execute the Kops script
```
. ./kops.sh
```
Retrieve information about the existing clusters
```
kops get cluster
```
Get information about the Kubernetes nodes in the cluster
```
kubectl get nodes
```
To scale out the node, execute the below command. Replace the `nodes-us-east-2a.mehar-2024-03-19-16-03.k8s.local` with the auto-scaling group (master or worker) in your case and also replace `region` with your region. MAke sure the Max capacity of the Auto Scaling Group is above or equal to the number you are trying to scale to.
```
aws autoscaling update-auto-scaling-group --auto-scaling-group-name nodes-us-east-2a.mehar-2024-03-19-16-03.k8s.local --desired-capacity 3 --region us-east-2
```
Run the below command if you are not able to retrieve the data. The below command comes in handy if you have downscaled your cluster and have scaled it up again. 
```
kops export kubeconfig --admin
```

### To delete the cluster
```
kops delete cluster --name <clustername> --state s3://<clustername>
```
