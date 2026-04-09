# 🚀 Kubernetes Part 1 – minikube & KOps Installation

This README serves as a **complete hands-on guide** for Kubernetes learning. It includes YAML manifests and command-line usage to help you practice kubernetes related topics.

## 📚 What You’ll Learn

- ✅ How to create a Production Grade Cluster using **kOps**
- ✅ How to create a Local **minikube** Cluster

---

## 🔧 Getting Started with kOps on AWS

[Click me](https://kops.sigs.k8s.io/getting_started/aws/) for Official Documentation of kOps

### 1️⃣ Install Kops for Linux

```bash
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

### 2️⃣ Install kubectl for Linux

```bash
curl -Lo kubectl https://dl.k8s.io/release/$(curl -s -L https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 3️⃣ Setup IAM user 

In order to build clusters within AWS we'll create a dedicated IAM user for kops. This user requires API credentials in order to use kops. Create the user, and credentials, using the AWS console.

The kops user will require the following IAM permissions to function properly:

```bash
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

### 4️⃣ Create S3 Bucket 

In order to store the state of your cluster, and the representation of your cluster, we need to create a dedicated S3 bucket for kops to use. This bucket will become the source of truth for our cluster configuration. In this guide we'll call this bucket ==**lwm-kubernetes-bucket**==, but you should add a custom prefix as bucket names need to be unique.

### 5️⃣ Prepare local environment 

You should record the SecretAccessKey and AccessKeyID in the returned JSON output, and then use them below:

```bash
# configure the aws client to use your new IAM user
aws configure           # Use your new access and secret key here
aws iam list-users      # you should see a list of all your IAM users here

# Create ssh-keys for kOps to use
ssh-keygen -t rsa -b 4096 -f ~/.ssh/kops-ssh-key
kops create secret --name <your-cluster-name> sshpublickey admin -i ~/.ssh/kops-ssh-key.pub

# Because "aws configure" doesn't export these vars for kops to use, we export them now
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
export NAME=myfirstcluster.k8s.local
export KOPS_STATE_STORE=s3://prefix-example-com-state-store
```

For a gossip-based cluster, make sure the name ends with ==**k8s.local.**==

### 6️⃣ kOps Commands

```bash
kops create cluster --name=${NAME} --cloud=aws --zones=ap-southeast-1a
kops edit cluster --name ${NAME}
kops update cluster --name ${NAME} --yes --admin
kops validate cluster
kops delete cluster --name ${NAME} --yes
```

---

## 🔧 Getting Started with minikube on AWS

[Click me](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download) for Official Documentation of minikube

### 1️⃣ Installation

Create a Linux EC2 VM with minimum  
- 2 CPUs or more 
- 2GB of free memory 
- 20GB of free disk space

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### 2️⃣ Start your cluster

```bash
minikube start
```

### 3️⃣ Interact with your cluster

```bash
minikube kubectl -- get pod -A
```

You can also make your life easier by adding the following to your shell config

```bash
alias kubectl="minikube kubectl --"
kubectl get pod
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 2 – YAML Reference Guide

In Part 2, you will learn how to create **Namespaces, Pods, ReplicaSets, DaemonSets.** Learn about kubectl cheat codes and both imperative and declarative way of deploying kubernetes Objects


## 📚 What You’ll Learn

- ✅ Creating and using **Namespaces**
- ✅ Defining **Pods** with one or more containers
- ✅ Using **ReplicaSets** to maintain pod availability
- ✅ Deploying **DaemonSets** to run a pod on each node

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ Namespace: qa

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: qa
```

### 2️⃣ Pod: myfirstpod (Single Container)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstpod
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
```

### 3️⃣ Pod: mysecondpod (Multi-Container: httpd + jenkins)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysecondpod
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
  - name: cont2
    image: jenkins/jenkins
    ports:
    - containerPort: 8080
```

### 4️⃣ Pod: mythirdpod (Multi-Container: httpd + nginx)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mythirdpod
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
  - name: cont2
    image: nginx
    ports:
    - containerPort: 80
```

### 5️⃣ Pod: myfirstpod in dev Namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstpod
  namespace: dev
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
```

### 6️⃣ ReplicaSet: lwm-replica (5 Replicas)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: lwm-replica
spec:
  # modify replicas according to your case
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd
```

### 7️⃣ DaemonSet: lwm-daemon

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: lwm-daemon
spec:
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd
```
---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 3 – YAML Reference Guide

In **Part 3**, you will learn how to manage applications using **Deployments** and expose them to users and other services using **Kubernetes Services**.


## 📚 What You’ll Learn

- ✅ Understand what a Deployment is and how it manages Pods  
- ✅ Create Deployments with different update strategies (RollingUpdate vs Recreate)  
- ✅ Expose Pods and Deployments using various Service types:
    - 🔹 **NodePort** – for accessing Pods externally via a fixed port on the node  
    - 🔹 **ClusterIP** – for internal Pod-to-Pod communication  
    - 🔹 **LoadBalancer** – for external access using cloud provider’s load balancer
- ✅ Demonstrate how **selectors and labels** work in real-world scenarios  
- ✅ Connect Pods internally and externally using appropriate Service types  
- ✅ Expose applications like **Jenkins** via NodePort  
- ✅ Test and validate connections using port mappings

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd
```

### 2️⃣ Deployment with Recreate Strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment-re
spec:
  strategy:
    type: Recreate
  replicas: 4
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd

```

### 3️⃣ Exposing a Pod with NodePort Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  labels:
    app: httpd
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-np-service
spec:
  type: NodePort
  selector:
    app: httpd
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30001
```

### 4️⃣ Exposing Jenkins Pod with NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-np-service
spec:
  type: NodePort
  selector:
    youtube: lwm
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30002
---
apiVersion: v1
kind: Pod
metadata:
  name: jenkins-pod
  labels:
    youtube: lwm
spec:
  containers:
  - name: cont1
    image: jenkins/jenkins
    ports:
    - containerPort: 8080
```

### 5️⃣ Pod-to-Pod Connection via ClusterIP Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    colour: black
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    colour: pink
spec:
  containers:
  - name: cont1
    image: nginx
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-cip-service
spec:
  type: ClusterIP
  selector:
    colour: pink
  ports:
    - port: 9999
      targetPort: 80
```

### 6️⃣ Exposing Deployment with LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-service
spec:
  type: LoadBalancer
  selector:
    colour: blue
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: blue
  template:
    metadata:
      labels:
        colour: blue
    spec:
      containers:
      - name: cont1
        image: httpd
```

### 7️⃣ Showcasing Selector and Labels with Deployment and Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: blue
  template:
    metadata:
      labels:
        colour: blue
    spec:
      containers:
      - name: cont1
        image: httpd
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    colour: blue
spec:
  containers:
  - name: cont1
    image: nginx
    ports:
    - containerPort: 80
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 4 – YAML Reference Guide

In **Part 4**, you will learn how to securely manage user access and permissions inside your EKS cluster using:

## 📚 What You’ll Learn

- ✅ `aws-auth` ConfigMap to add IAM users and roles  
- ✅ Role-Based Access Control (RBAC) for namespace-specific and cluster-wide access  
- ✅ Create `Role`, `RoleBinding`, `ClusterRole`, and `ClusterRoleBinding`  
- ✅ Map IAM users to Kubernetes RBAC groups  
- ✅ Use **IRSA (IAM Roles for Service Accounts)** to allow Pods to access AWS services like S3 securely  
- ✅ Hands-on live demo of a Pod accessing S3 using IRSA

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ aws-auth ConfigMap – Adding IAM Users

```yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::992382429239:role/eks-node-group-role
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::992382429239:user/dev-user
      username: dev-user
      groups:
        - dev-group
    - userarn: arn:aws:iam::992382429239:user/readonly-user
      username: readonly-user
      groups:
        - readonly-group
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
```

### 2️⃣ Dev User Role and RoleBinding (Namespace-scoped Access)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-role
  namespace: dev-namespace
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "services", "deployments", "jobs"]
  verbs: ["get", "list", "create", "update", "delete", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-binding
  namespace: dev-namespace
subjects:
- kind: Group
  name: dev-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
```

### 3️⃣ ReadOnly User ClusterRole and ClusterRoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: readonly-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "services", "deployments", "jobs"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: readonly-binding
subjects:
- kind: Group
  name: readonly-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: readonly-role
  apiGroup: rbac.authorization.k8s.io
```

### 4️⃣ Pod Accessing S3 Using IRSA

#### 🔹 Step 1: Enable OIDC Provider for Your EKS Cluster

```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-southeast-1 \
  --cluster <cluster-name> \
  --approve
```

#### 🔹 Step 2: Create IAM Role with Trust Policy for Pod
#### 🔹 Step 3: Create Kubernetes ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-access
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/EKS_S3_IRSA_Role
```

#### 🔹 Step 4: Deploy a Pod Using the Service Account

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: s3-reader
spec:
  serviceAccountName: s3-access
  containers:
  - name: awscli
    image: amazonlinux
    command: ["/bin/sh"]
    args: ["-c", "yum install -y aws-cli && aws s3 ls"]
```

#### 🔹 Step 5: Verify Access to S3 from Pod

```bash
2025-06-01 00:00:00 lwm-terraform-bucket
2025-06-01 00:00:00 lwm-kubernetes-bucket
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 5 – YAML Reference Guide

In **Part 5**, you will learn advanced pod scheduling techniques in Kubernetes to control how and where your Pods run within your EKS cluster.

## 📚 What You’ll Learn

 - ✅ **Node Selector** – Assign Pods to specific nodes using simple labels
 - ✅ **Node Affinity** – Fine-grained control over Pod placement using label expressions
 - ✅ **Pod Affinity & Anti-Affinity** – Co-locate or separate Pods based on labels and topology
 - ✅ **Taints and Tolerations** – Ensure only specific Pods are scheduled on tainted nodes
 - ✅ Use **requiredDuringSchedulingIgnoredDuringExecution** and understand its impact
 - ✅ Real-world examples with **multiple tolerations** (e.g., green, blue)
 - ✅ Hands-on live demo with YAML files showing each scheduling strategy in action

---

## 🛠️ Cluster Setup: Three Worker Nodes in EKS

We will create three EKS worker nodes with custom labels and taints.

### ✅ Step-by-Step

1. Launch EKS cluster with 3 managed node groups (use console)
2. After nodes are ready, label and taint the nodes using kubectl: (in video the taint and labels are added in console you can also use the below commands)

```bash
# Label node 1 as general
kubectl label node <node-name-1> node-type=general

# Label and taint node 2 as high-cpu with taint colour=green
kubectl label node <node-name-2> node-type=high-cpu
kubectl taint node <node-name-2> colour=green:NoSchedule

# Label and taint node 3 as high-memory with taint colour=blue
kubectl label node <node-name-3> node-type=high-memory
kubectl taint node <node-name-3> colour=blue:NoSchedule
```

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ Basic Pod on Any Node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### 2️⃣ Pod with Required Node Affinity (high-memory)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-intensive-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-type
            operator: In
            values:
            - high-memory
  containers:
  - name: memory-app
    image: nginx
```

### 3️⃣ Pod with Preferred Node Affinity (high-memory)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-intensive-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: node-type
            operator: In
            values:
            - high-memory
  containers:
  - name: memory-app
    image: nginx
```

### 4️⃣ Pod with Toleration for Taint (colour=green:NoSchedule)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-intensive-pod
spec:
  tolerations:
  - key: "colour"
    operator: "Equal"
    value: "green"
    effect: "NoSchedule"
  containers:
  - name: cpu-app
    image: nginx
```

### 5️⃣ Deployment Tolerating Green Taint

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-app-deployment
  labels:
    app: green-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: green-app
  template:
    metadata:
      labels:
        app: green-app
    spec:
      tolerations:
      - key: "colour"
        operator: "Equal"
        value: "green"
        effect: "NoSchedule"
      containers:
      - name: green-container
        image: nginx
        ports:
        - containerPort: 80
```

### 6️⃣ Deployment Tolerating Both green and blue Taints

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multicolor-app
  labels:
    app: multicolor-app
spec:
  replicas: 8
  selector:
    matchLabels:
      app: multicolor-app
  template:
    metadata:
      labels:
        app: multicolor-app
    spec:
      tolerations:
      - key: "colour"
        operator: "Equal"
        value: "green"
        effect: "NoSchedule"
      - key: "colour"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
      containers:
      - name: multicolor-container
        image: nginx
        ports:
        - containerPort: 80
```

### 7️⃣ Deployment with Node Affinity and Toleration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: compute-app-deployment
  labels:
    app: compute-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: compute-app
  template:
    metadata:
      labels:
        app: compute-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-type
                operator: In
                values:
                - high-cpu
      tolerations:
      - key: "colour"
        operator: "Equal"
        value: "green"
        effect: "NoSchedule"
      containers:
      - name: compute-container
        image: nginx
        ports:
        - containerPort: 80
```

### 8️⃣ Pod with Label for Pod Affinity Targeting

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    group: bestfriendz 
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### 9️⃣ Pod Affinity (Schedule with bestfriendz Pods)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            group: bestfriendz
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: cont1
    image: httpd
```

### 1️⃣0️⃣ Pod Anti-Affinity (Avoid bestfriendz Pods)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: enemy-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            group: bestfriendz
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: cont1
    image: httpd
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 6 – YAML Reference Guide

In **part 6**, you'll explore how to persist and share data between Pods using different types of volumes. You’ll also understand how to dynamically and statically provision storage in AWS with EBS and EFS.

## 📚 What You’ll Learn

- ✅ Difference between `emptyDir`, `hostPath`, `PersistentVolume` (PV) and `PersistentVolumeClaim` (PVC)  
- ✅ Use `StorageClass` for dynamic provisioning of **EBS** and **EFS**  
- ✅ Understand how `ReadWriteOnce` and `ReadWriteMany` access modes affect Pod mounting  
- ✅ Share data between containers using `emptyDir`  
- ✅ Mount host machine files into Pods using `hostPath`  
- ✅ Use **EBS** volumes with static and dynamic provisioning  
- ✅ Use **EFS** for shared, scalable, and dynamic file storage  
- ✅ Full hands-on examples with YAML

---

### 🛠️ Cluster Setup: Enable the Add-on with Pod Identity (via Console)

- 1️⃣ Go to EKS > Your Cluster > Add-ons
- 2️⃣ Click Create Add-on
- 3️⃣ Choose aws-ebs-csi-driver & aws-efs-csi-driver
- 4️⃣ Select latest version
- 5️⃣ For IAM Role, choose:
- 6️⃣ Pod Identity (recommended)
- 7️⃣ Select or create the IAM role: AmazonEKS_EBS_CSI_DriverRole, AmazonEKS_EFS_CSI_DriverRole

---

# 📦 Kubernetes Volumes – Quick Notes

## ⚠️ Pods are Ephemeral!
 ==When a Pod is deleted, everything inside it is lost — unless it's backed by a volume.==

---

## 📁 Volume Types

| Volume Type   | Description                                             |
|---------------|---------------------------------------------------------|
| `emptyDir`    | Temporary storage shared between containers in a Pod. Deleted when the Pod is deleted. |
| `hostPath`    | Mounts a file or directory from the host node into the Pod. **Not recommended** for production use. |
| **Persistent Volumes** | Backed by real storage from cloud or local systems. Persist beyond Pod lifecycle. |
| ├─ EBS         | Amazon Elastic Block Store — supports **RWO** access. |
| └─ EFS         | Amazon Elastic File System — supports **RWX** access. |

---

## ⚙️ Provisioning Modes

| Mode               | Description                                              |
|--------------------|----------------------------------------------------------|
| **Static**         | Volume is pre-created manually (e.g., via AWS Console)   |
| **Dynamic**        | Volume is created **automatically** using a `StorageClass` |

---

## 🔓 Access Modes

| Access Mode         | Description                                           | Example Volume |
|---------------------|-------------------------------------------------------|----------------|
| `ReadWriteOnce` (RWO) | One node can read/write at a time                   | EBS            |
| `ReadOnlyMany` (ROX)  | Many nodes can read, but none can write             | EFS (rare)     |
| `ReadWriteMany` (RWX) | Many nodes can read/write simultaneously            | EFS            |
| `ReadWriteOncePod`    | Only a single pod (on any node) can access the volume | Rare use case  |

---

## ♻️ Reclaim Policies

| Policy     | Description                                                      |
|------------|------------------------------------------------------------------|
| `Retain`   | Manual cleanup required. Volume and data are retained.          |
| `Recycle`  | Performs basic scrub (`rm -rf /volume/*`) — deprecated in many environments |
| `Delete`   | Deletes the associated storage resource automatically (e.g., EBS volume) |

---

## 🔁 volumeBindingMode

| Mode                  | Behavior                                                              |
|-----------------------|-----------------------------------------------------------------------|
| `WaitForFirstConsumer` | Volume is created **only when Pod is scheduled** (prevents cross-AZ issues) |
| `Immediate`           | Volume is created **as soon as PVC is created**, even before pod uses it |

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ Sharing Data Between Containers Using `emptyDir`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-example
spec:
  containers:
    - name: writer-cont1
      image: busybox
      command: ["/bin/sh", "-c", "echo 'Hello from writer' > /data/message.txt && sleep 3600"]
      volumeMounts:
        - mountPath: /data
          name: shared-data
    - name: reader-cont2
      image: busybox
      command: ["/bin/sh", "-c", "cat /data/message.txt && sleep 3600"]
      volumeMounts:
        - mountPath: /data
          name: shared-data
  volumes:
    - name: shared-data
      emptyDir: {}
```

### 2️⃣ Accessing Host Files Using hostPath

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-etc-example
spec:
  containers:
    - name: etc-reader
      image: busybox
      command: ["/bin/sh", "-c", "ls /host/etc && sleep 3600"]
      volumeMounts:
        - mountPath: /host/etc
          name: host-etc
  volumes:
    - name: host-etc
      hostPath:
        path: /etc
        type: Directory
```

### 3️⃣ Dynamic EBS Provisioning Using StorageClass and PVC

```yaml
# StorageClass for EBS
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
---
# PVC for EBS
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
  storageClassName: ebs-sc
---
# Pod using the dynamic EBS volume
apiVersion: v1
kind: Pod
metadata:
  name: ebs-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: ebs-volume
  volumes:
  - name: ebs-volume
    persistentVolumeClaim:
      claimName: ebs-pvc

```

### 4️⃣ Static EBS Volume with Manual PV and PVC

```yaml
# Static PersistentVolume (replace with actual EBS Volume ID)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-ebs-pv
spec:
  capacity:
    storage: 3Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  csi:
    driver: ebs.csi.aws.com
    volumeHandle: vol-xxxxxxxxxxxxxx     # Replace EBS Volume id
---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-ebs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 3Gi
---
# Pod using the static EBS volume
apiVersion: v1
kind: Pod
metadata:
  name: ebs-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: ebs-volume
  volumes:
  - name: ebs-volume
    persistentVolumeClaim:
      claimName: static-ebs-pvc

```

### 5️⃣ Dynamic EFS Volume Provisioning with StorageClass and PVC

```yaml
# StorageClass for EFS
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-0f8431a1f00634456  # Replace with your EFS ID
  directoryPerms: "700"
  gidRangeStart: "1000"
  gidRangeEnd: "2000"
  basePath: "/dynamic_provisioning"
---
# PVC for EFS
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: efs-sc
---
# Pod using EFS volume
apiVersion: v1
kind: Pod
metadata:
  name: efs-test-pod
spec:
  containers:
    - name: app
      image: busybox
      command: [ "/bin/sh", "-c", "echo EFS volume is working > /data/hello.txt && sleep 3600" ]
      volumeMounts:
        - name: efs-volume
          mountPath: /data
  volumes:
    - name: efs-volume
      persistentVolumeClaim:
        claimName: efs-pvc
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚦 Kubernetes Part 7 – Probes, Init Containers, Sidecars & Environment Variables


## 📚 What You Will Learn

In **Day 7**, you'll learn how Kubernetes manages **application health**, **startup sequences**, and **shared responsibilities** between containers. This session is packed with **live examples** and advanced techniques.

✅ Understand how **Liveness**, **Readiness**, and **Startup** probes work  
✅ Configure probes to detect failures, slow startups, and readiness checks  
✅ Use **Init Containers** for setup tasks before app starts  
✅ Implement the **Sidecar pattern** to enhance or monitor your main container  
✅ Inject configuration using **Environment Variables**, **ConfigMaps**, and **Secrets**  
✅ Mount secrets as files and access securely

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 🧪 1️⃣ Pod example with All Three Probes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: demo-app
    image: httpd:latest
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 3
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
``` 

## ⏱️ Probe Timing Example (Understanding Failure/Success)

Let's say you have this probe configuration:

```yaml
initialDelaySeconds: 15
periodSeconds: 30
failureThreshold: 4
successThreshold: 2
timeoutSeconds: 10
```

| Time       | Event               | Result            |
| ---------- | ------------------- | ----------------- |
| 2:00:00 PM | Pod Started         |                   |
| 2:00:15 PM | 1st Check (Fail)    | ❌                 |
| 2:00:45 PM | 2nd Check (Fail)    | ❌                 |
| 2:01:15 PM | 3rd Check (Fail)    | ❌                 |
| 2:01:45 PM | 4th Check (Fail)    | ❌ Pod Restart     |
| 2:02:15 PM | 5th Check (Success) | ✅                 |
| 2:02:45 PM | 6th Check (Success) | ✅ (Healthy again) |

> 💡 Note: The probe starts only after the initialDelaySeconds. If failureThreshold is reached with consecutive failures, the container will be restarted. After restart, the container must pass successThreshold consecutive checks to be marked as Ready.

### 🧪 2️⃣ Liveness Probe with NGINX

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 3

```

### 🧪 3️⃣ Liveness Probe with Exec (BusyBox)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-busybox
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "touch /tmp/healthy && sleep 3600"]
    livenessProbe:
      exec:
        command: ["cat", "/tmp/healthy"]
      initialDelaySeconds: 5
      periodSeconds: 10
      failureThreshold: 3

```

### 🏁 4️⃣ Init Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo "App started!" && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'echo "Initializing..."; sleep 5']
```

### 🤝 5️⃣ Sidecar Pattern – Logging Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo "Writing log..." && while true; do echo log >> /shared/log.txt; sleep 5; done']
    volumeMounts:
    - name: shared-logs
      mountPath: /shared

  - name: log-watcher
    image: busybox
    command: ['sh', '-c', 'tail -f /shared/log.txt']
    volumeMounts:
    - name: shared-logs
      mountPath: /shared

  volumes:
  - name: shared-logs
    emptyDir: {}
```

### 🌿 6️⃣ Using Environment Variables  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: demo
    image: busybox
    command: ["sh", "-c", "echo $ENV_MSG && sleep 3600"]
    env:
    - name: ENV_MSG
      value: "Hello from LearnWithMithran Youtube!"
```

### ⚙️ 7️⃣ ConfigMap and Secret as envFrom

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLR: blue
  APP_MODE: prod
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  Channel: TGVhcldpdGhNaXRocmFuCg==
  Owner: TWl0aHJhbgo=
---
apiVersion: v1
kind: Pod
metadata:
  name: simple-webcolor-pod
spec:
  containers:
  - name: simple-webcolor-pod
    image: httpd
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secret
```

### 🔐 8️⃣ Secrets as Environment Variable and Mounted File

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
data:
  password: dXBkYXRlZDQ1Ngo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: demo
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo ENV: $PASSWORD; echo FILE: $(cat /etc/secret/password); sleep 10; done"]
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: demo-secret
          key: password
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: demo-secret
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 8 – AWS EKS ALB Ingress Controller


## 📚 What You Will Learn

In **Day 8**, you'll learn how to configure advanced Kubernetes networking and traffic routing using the **AWS ALB Ingress Controller** and **Helm**. This session helps you build real-world multi-team infrastructure in an Amazon EKS cluster.

✅ Install and configure eksctl and Helm for production-grade clusters
✅ Deploy the AWS Load Balancer Controller using Helm and IAM integration
✅ Create and expose applications using Kubernetes Ingress
✅ Set up path-based routing with a shared ALB across teams
✅ Deploy workloads in isolated namespaces (e.g., team-a, team-b)
✅ Use annotations to control Ingress behavior, grouping, and routing
✅ Understand the difference between LoadBalancer and Ingress services
✅ Deploy multi-container apps and manage traffic securely and efficiently

---

## ❗ Why Not Use LoadBalancer Services Alone?

Using Kubernetes Service of type LoadBalancer works well for exposing single services, but it has major limitations:

### ⚠️ Drawbacks of LoadBalancer Service

- 🚫 One ALB per Service → costly in production
- 📈 Scales poorly if you have many microservices
- 🧱 No support for path-based or host-based routing
- 🧍‍♂️ Harder to manage for multiple teams/projects
- 🔐 Difficult to apply centralized security policies

## 🧪 Example Issue with LoadBalancer Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-service-1
spec:
  type: LoadBalancer
  selector:
    colour: blue
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment-1
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: blue
  template:
    metadata:
      labels:
        colour: blue
    spec:
      containers:
      - name: cont1
        image: httpd
---
apiVersion: v1
kind: Service
metadata:
  name: lb-service-2
spec:
  type: LoadBalancer
  selector:
    colour: black
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment-2
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: black
  template:
    metadata:
      labels:
        colour: black
    spec:
      containers:
      - name: cont1
        image: nginx
```

## ✅ Why Use Ingress with ALB Controller?

Ingress with ALB controller provides:

- ✅ Single ALB for multiple apps (shared ALB)
- 🔁 Path-based routing (/app1, /app2, etc.)
- 🧑‍🤝‍🧑 Team-level separation via namespaces & ingress group
- 💸 Cost savings – fewer ALBs created
- 🔐 Centralized control over routing & TLS termination
- ☁️ Better AWS-native integration via Load Balancer Controller

> ✅ Recommendation: Use Ingress + AWS Load Balancer Controller for scalable, secure, and cost-efficient EKS routing.


## ⚙️ Prerequisites to Install ALB Ingress Controller

- AWS CLI configured
- IAM user with appropriate EKS permissions
- `kubectl` installed
- `eksctl` and `helm` installed

### 🧱 eksctl Setup

```bash
# Install eksctl
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```

### 🔐 OIDC and IAM Setup

```bash
eksctl utils associate-iam-oidc-provider \
  --region=ap-southeast-1 \
  --cluster=LearnWithMithran2 \
  --approve
```

### 📥 Download the IAM policy:

[Click Me for IAM Json Policy](https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/refs/heads/main/docs/install/iam_policy.json)

Create the IAM policy in AWS as: `AWSLoadBalancerControllerIAMPolicy2`

### 📥 Create service account:

```bash
eksctl create iamserviceaccount \
  --cluster LearnWithMithran2 \
  --region ap-southeast-1 \
  --namespace kube-system \
  --name aws-load-balancer-controller2 \
  --attach-policy-arn arn:aws:iam::992382429239:policy/AWSLoadBalancerControllerIAMPolicy2 \
  --approve
```

### 📦 Helm Setup

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### 📡 Add and update repo:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

### 🚀 Install AWS ALB Ingress Controller

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=LearnWithMithran \
  --set region=ap-southeast-1 \
  --set vpcId=vpc-020d3da5a9615c591 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller2
```

---

## 🌐 Ingress Setup: Basic Echo App


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: hashicorp/http-echo
        args: ["-text=Hello from app"]
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello
  ports:
  - port: 80
    targetPort: 5678
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
  - http:
      paths:
      - path: /LWM
        pathType: Prefix
        backend:
          service:
            name: hello-service
            port:
              number: 80
```

## 👥 Multi-Team Namespaces with Shared ALB

### Create Namespaces

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-b
```

### Team A

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-a
  namespace: team-a
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-a
  template:
    metadata:
      labels:
        app: app-a
    spec:
      containers:
      - name: app-a
        image: hashicorp/http-echo
        args: ["-text=Hello from App A LWM"]
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app-a-svc
  namespace: team-a
spec:
  selector:
    app: app-a
  ports:
  - port: 80
    targetPort: 5678
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-a-ingress
  namespace: team-a
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/group.name: shared-alb
spec:
  rules:
  - http:
      paths:
      - path: /a
        pathType: Prefix
        backend:
          service:
            name: app-a-svc
            port:
              number: 80
```

### Team B
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-b
  namespace: team-b
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-b
  template:
    metadata:
      labels:
        app: app-b
    spec:
      containers:
      - name: app-b
        image: hashicorp/http-echo
        args: ["-text=Hello from App B LWM"]
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app-b-svc
  namespace: team-b
spec:
  selector:
    app: app-b
  ports:
  - port: 80
    targetPort: 5678
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-b-ingress
  namespace: team-b
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/group.name: shared-alb
spec:
  rules:
  - http:
      paths:
      - path: /b
        pathType: Prefix
        backend:
          service:
            name: app-b-svc
            port:
              number: 80
```

## 📌 Notes

- Ingress class should match your ALB controller: kubernetes.io/ingress.class: alb
- Shared ALB group enables multi-team routing via paths
- Ensure VPC ID and IAM policies are correctly set in the helm install step

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 9 – Jobs, CronJobs, StatefulSets, HPA, and ExternalName Services


## 📚 What You Will Learn

In **Day 9**, we deep dive into *Kubernetes workload types and resource management*. These examples are hands-on and ideal for mastering real-world use cases with **Jobs, CronJobs, StatefulSets, ExternalName, Resource Requests/Limits, and Horizontal Pod Autoscaling (HPA)**.

✅ Create one-time Jobs and scheduled CronJobs
✅ Understand StatefulSet and Headless Services (with and without volumes)
✅ Simulate FQDN with ExternalName services
✅ Apply resource requests and limits to containers
✅ Configure HPA using the Kubernetes Metrics Server
✅ Deploy multi-replica applications using Deployments and Services
✅ Run test pods and simulate DNS-based access

---

## 🛠️ Kubernetes Job and CronJob – One-time and Recurring Tasks

Jobs and CronJobs help run short-lived or recurring background tasks in Kubernetes.

### 🔨 What is a Job?
A Job creates one or more pods and ensures that a specific task runs to completion. Once the task finishes successfully, the Job is considered complete.

✅ Use when you need to:

- Run a one-time task
- Perform data migration or setup scripts
- Process a batch job or backup operation

#### 🔧 Example YAML – Job

```yaml
# One-time Job
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello from Kubernetes Job"]
      restartPolicy: Never
  backoffLimit: 2
```


### ⏰ What is a CronJob?
A CronJob creates Jobs on a schedule, like a traditional Linux cron. It’s ideal for repeating tasks such as log rotation, cleanup, backups, etc.

✅ Use when you need to:

- Run a task at specific time intervals
- Automate recurring operations
- Perform periodic cleanups or reports

#### 🔧 Example YAML – CronJob

```yaml
# Recurring CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-task
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: busybox
            command: ["sh", "-c", "echo Cleaning temp files at $(date)"]
          restartPolicy: OnFailure
```

### 🆚 Jobs vs CronJobs – Comparison Table

| Feature            | **Job**                                     | **CronJob**                       |
| ------------------ | ------------------------------------------- | --------------------------------- |
| **Run Type**       | One-time execution                          | Recurring on schedule             |
| **Scheduling**     | Immediate or manual                         | CRON-based (e.g., every 5 mins)   |
| **Use Case**       | Backup, data migration, processing one file | Daily reports, cleanup tasks      |
| **Pod Lifecycle**  | Runs once then terminates                   | Triggers Job objects at intervals |
| **Built-in Retry** | ✅ Yes (`backoffLimit`)                      | ✅ Yes, inherited from Job spec    |


---

## ⚔️ Deployments vs StatefulSets

Kubernetes provides different workload types for managing application lifecycles. Two of the most common are Deployments and StatefulSets.


### ✅ When to Use Deployments

- Your application is stateless
- You want easy horizontal scaling
- Any pod can serve any request (e.g., web servers, REST APIs, frontend apps)
- Persistent storage is not required or shared storage is enough

### ✅ When to Use StatefulSets

- Each pod needs a stable identity
- You want stable persistent storage per pod
- Pods need stable DNS names
- You require ordered, graceful deployment, scaling, or deletion
- Use cases include databases (MySQL, MongoDB), queues, Kafka, Zookeeper

### 🔍 What’s the Difference?

| Feature                    | **Deployment**                                              | **StatefulSet**                                                    |
| -------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------ |
| **Pod Name**               | Dynamic, auto-generated (e.g., `app-xyz123`)                | Stable and predictable (`app-0`, `app-1`)                          |
| **Pod Identity**           | All pods are identical (no unique identity)                 | Each pod has a **unique, stable identity**                         |
| **Storage**                | Shared or ephemeral storage                                 | Dedicated persistent volume per pod                                |
| **Network Identity (DNS)** | Shared service FQDN (e.g., `svc.default.svc.cluster.local`) | Unique FQDN per pod (e.g., `pod-0.svc.default.svc.cluster.local`)  |
| **Scaling Behavior**       | All pods are interchangeable                                | Pods are created **sequentially** and deleted in **reverse order** |
| **Use Case**               | Stateless applications                                      | Stateful applications                                              |


## 🧱 StatefulSet With Headless Service (Without Volumes)

```yaml
# Headless Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
    name: http
---
# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx-headless"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

## 🧱 Deployment With ClusterIp Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: blue
  template:
    metadata:
      labels:
        colour: blue
    spec:
      containers:
      - name: cont1
        image: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    colour: blue
  ports:
  - port: 80
    targetPort: 80
```

## 🔄 StatefulSet vs Deployment – FQDN Behavior and Identity

When running microservices in Kubernetes, how services are addressed (FQDN) matters for stable communication, especially in distributed systems like databases or stateful workloads.

| Feature                 | **Deployment**                                 | **StatefulSet**                                                  |
| ----------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| **Pod Name**            | Random (e.g., `httpd-deploy-5f89dfd9cc-abcde`) | Fixed (e.g., `nginx-0`, `nginx-1`, `nginx-2`)                    |
| **ClusterIP**           | Assigned (e.g., `10.100.247.159`)              | `None` (Headless Service)                                        |
| **Pod IP**              | Dynamic (changes on restart)                   | Dynamic (still can change, but FQDN is stable)                   |
| **FQDN Format**         | `httpd-svc.default.svc.cluster.local`          | `nginx-0.nginx-headless.default.svc.cluster.local`, `nginx-1...` |
| **Stable DNS Identity** | ❌ No stable per-pod DNS                        | ✅ Stable and addressable via predictable FQDN                    |
| **Use Case**            | Stateless apps (e.g., web servers)             | Stateful apps (e.g., DBs, queues)                                |

## 📘 Example: FQDN Usage

```
# Deployment
FQDN → httpd-svc.default.svc.cluster.local
Pod IP → Changes on restart (e.g., 172.31.4.118)

# StatefulSet
FQDNs →
- nginx-0.nginx-headless.default.svc.cluster.local
- nginx-1.nginx-headless.default.svc.cluster.local
- nginx-2.nginx-headless.default.svc.cluster.local

Pod IP → May change, but name-based resolution is stable
```

## 🧪 Testing DNS FQDN from a dummy Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-test
spec:
  containers:
  - name: test
    image: curlimages/curl
    command: ["sleep", "3600"]
```

## 📦 StatefulSet With Persistent Volumes ==(need PV & PVC)==

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - port: 6379
    name: redis
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: "redis-headless"
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi

```

---

## 🌐 Kubernetes ExternalName Service – Access External Resources via DNS

### 🔍 What is an ExternalName Service?
An ExternalName service is a special type of Kubernetes Service that allows pods inside the cluster to access an external service (outside the cluster) using an internal DNS name.

Instead of routing traffic to internal pods or endpoints, it returns a CNAME (DNS alias) pointing to an external FQDN.

### 🔧 Example Use Cases

✅ Allow applications to access:
- An external database (e.g., RDS, MongoDB Atlas)
- A legacy app running outside Kubernetes (e.g., on EC2)
- Third-party APIs or SaaS systems (e.g., Google, Stripe)

#### 🛠️ Sample YAML – ExternalName to EC2 Endpoint

```yaml
# EC2-hosted app
apiVersion: v1
kind: Service
metadata:
  name: lwm-svc
spec:
  type: ExternalName
  externalName: ec2-52-77-254-161.ap-southeast-1.compute.amazonaws.com
```

#### 🔧 Sample YAML – ExternalName to Public Website (Google)

```yaml
# Google ExternalName (for test/demo only)
apiVersion: v1
kind: Service
metadata:
  name: google-svc
spec:
  type: ExternalName
  externalName: www.google.com
```

---

## ⚙️ Kubernetes Resource Requests and Limits – Manage Pod Resource Usage

### 🔍 What Are Requests and Limits?
Kubernetes allows you to control how much CPU and memory a container can request and use through requests and limits. This helps ensure fair resource allocation, avoid overconsumption, and enable auto-scaling.

### 🧠 Key Concepts

| Term           | Meaning                                                                |
| -------------- | ---------------------------------------------------------------------- |
| **Request**    | The **minimum** amount of CPU/memory guaranteed to the container       |
| **Limit**      | The **maximum** amount of CPU/memory the container is allowed to use   |
| **Eviction**   | If a pod exceeds limits or system is under pressure, it may be evicted |
| **Throttling** | CPU usage above limit is throttled (not evicted)                       |

#### 🔧 Example YAML – Resource Request and Limit

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-rl
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

#### 💡 CPU Units Explained

| Value  | Meaning         |
| ------ | --------------- |
| `1`    | 1 full CPU core |
| `500m` | 0.5 CPU core    |
| `100m` | 0.1 CPU core    |


#### 💡 Memory Units
Common formats: Mi, Gi, M, G

Example: 128Mi = 128 Mebibytes

### ⚙️ What Happens at Runtime?
- If the node runs out of resources:
  - Pods may be evicted if they exceed memory limits
  - CPU usage will be throttled if it exceeds CPU limits
- Requests help the scheduler decide where to place pods

### 🧪 Real-World Use Case

| Scenario                   | Request              | Limit                |
| -------------------------- | -------------------- | -------------------- |
| Small API                  | 100m CPU / 128Mi RAM | 200m CPU / 256Mi RAM |
| Heavy Worker               | 500m CPU / 512Mi RAM | 1 CPU / 1Gi RAM      |
| Memory-intensive Batch Job | 256Mi RAM            | 1Gi RAM              |

---

## 📈 HPA vs VPA – Kubernetes Auto Scaling Demystified
In Kubernetes, auto-scaling ensures that your application has the right number of resources and replicas based on real-time demand. The two main types of autoscalers are:

- **Horizontal Pod Autoscaler (HPA)** – scales the number of pod replicas
- **Vertical Pod Autoscaler (VPA)** – adjusts the resource requests/limits for each pod

### 🔄 What is HPA (Horizontal Pod Autoscaler)?
HPA automatically adjusts the number of pods in a Deployment, ReplicaSet, or StatefulSet based on observed CPU/memory usage or custom metrics.

### ✅ Use when:

- You want to scale out/in based on traffic/load
- Your app is stateless
- Metrics are available via Metrics Server

#### 🔧 Example YAML – HPA

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
---
# HPA (Metrics Server Required)
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

### 🧱 What is VPA (Vertical Pod Autoscaler)?
VPA automatically adjusts the CPU and memory requests/limits of containers in a pod to better match the actual usage.

### ✅ Use when:

- You want to optimize resource usage per pod
- Your app does not scale well horizontally
- You need to reduce over-provisioning

#### 🔧 Modes in VPA:

| Mode      | Behavior                         |
| --------- | -------------------------------- |
| `Off`     | Just monitor; no action          |
| `Auto`    | Actively updates pod resources   |
| `Initial` | Only sets values on pod creation |

#### Example YAML – VPA

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: php-apache-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       php-apache
  updatePolicy:
    updateMode: "Auto"
```

---

# 🚀 Kubernetes Part 10 – CRI, CNI & CSI – The Core Kubernetes Interfaces


## 📚 What You Will Learn

In **Day 10**, we explore the **three critical plugin interfaces** (CRI, CNI, CSI), **Network Policies**, **Pod Security Admission**, and how to perform a **Kubernetes Cluster Upgrade** safely.

✅ Understand what CRI, CNI, and CSI are and why they exist
✅ Learn how Kubernetes decouples runtime, network, and storage via interfaces
✅ Explore popular plugins for each interface
✅ Hands-on Network Policy lab – deny all traffic and allow from specific namespaces
✅ Enforce Pod Security Standards using Pod Security Admission (PSA)
✅ Step-by-step Kubernetes cluster upgrade (EKS 1.34 → 1.35)
✅ Use PodDisruptionBudgets, kubectl drain, cordon, and uncordon

---

## 🧩 Overview – Why CRI, CNI & CSI?

Kubernetes follows a **plugin-based architecture**. Instead of tightly coupling container runtimes, networking, and storage into the core, Kubernetes defines **standard interfaces** that allow third-party providers to plug in their own implementations.

| Interface | Full Form | Responsibility |
| --------- | ------------------------------ | ---------------------------------- |
| **CRI** | Container Runtime Interface | Managing container lifecycle |
| **CNI** | Container Network Interface | Pod-to-Pod networking |
| **CSI** | Container Storage Interface | Provisioning and managing storage |

---

## 🏗️ CRI – Container Runtime Interface

### 🔨 What is CRI?

CRI is the **standard API** that Kubernetes uses to communicate with container runtimes. It defines how the kubelet interacts with the runtime to pull images, create containers, start/stop them, and collect logs.

Before CRI, Kubernetes was tightly coupled to Docker. CRI was introduced to make the runtime layer **pluggable and interchangeable**.

### 🔄 How CRI Works

```
kubelet  ──(gRPC)──▶  CRI Runtime Shim  ──▶  Container Runtime (e.g., containerd, CRI-O)
```

The kubelet talks to the CRI shim via **gRPC**, and the shim translates the requests into runtime-specific calls.

### ✅ Popular CRI Runtimes

| Runtime | Description |
| --------------- | -------------------------------------------------- |
| **containerd** | Default runtime for most Kubernetes distributions |
| **CRI-O** | Lightweight runtime designed specifically for K8s |
| **kata-runtime**| Runs containers inside lightweight VMs for isolation|

### 🔧 Check Your Current CRI Runtime

```bash
# Check the container runtime on each node
kubectl get nodes -o wide
```

The `CONTAINER-RUNTIME` column shows which runtime is in use (e.g., `containerd://1.7.x`).

### 📌 Key CRI Concepts

- **Image Service** – Pulls, lists, and removes container images
- **Runtime Service** – Creates, starts, stops, and removes containers
- **PodSandbox** – Manages the pod-level isolation (network namespace, etc.)

---

## 🌐 CNI – Container Network Interface

### 🔨 What is CNI?

CNI is the **standard interface** for configuring networking in Linux containers. In Kubernetes, the CNI plugin is responsible for:

- Assigning IP addresses to Pods
- Setting up network routes between Pods across nodes
- Configuring network namespaces

### 🔄 How CNI Works

```
Pod Created  ──▶  kubelet calls CRI  ──▶  CRI calls CNI plugin  ──▶  Network configured for Pod
```

When a Pod is scheduled, the kubelet asks the container runtime to create a sandbox. The runtime invokes the configured CNI plugin to **set up networking** for that Pod.

### ✅ Popular CNI Plugins

| Plugin | Key Features |
| ------------- | ------------------------------------------------------- |
| **Calico** | Network policies, BGP routing, VXLAN/IPIP overlay |
| **Flannel** | Simple VXLAN overlay, easy to set up |
| **Weave Net** | Mesh networking, encryption support |
| **Cilium** | eBPF-based, advanced security & observability |
| **AWS VPC CNI**| Native AWS VPC networking for EKS pods |

### 📌 Key CNI Concepts

- **Pod CIDR** – The IP range assigned to Pods in the cluster
- **Overlay Network** – Virtual network (VXLAN/IPIP) that spans across nodes
- **Network Policy** – Rules that control ingress/egress traffic between Pods
- **IPAM** – IP Address Management for assigning IPs to Pods

### 🔍 CNI Plugin Comparison

| Feature | Calico | Flannel | Cilium | Weave Net |
| ------------------- | ------ | ------- | ------ | --------- |
| Network Policies | ✅ | ❌ | ✅ | ✅ |
| Encryption | ✅ | ❌ | ✅ | ✅ |
| eBPF Support | ✅ | ❌ | ✅ | ❌ |
| Ease of Setup | Medium | Easy | Medium | Easy |
| Performance | High | Medium | High | Medium |

---

## 💾 CSI – Container Storage Interface

### 🔨 What is CSI?

CSI is the **standard interface** for exposing storage systems to containerized workloads in Kubernetes. It allows storage vendors to develop plugins **once** and have them work across any container orchestrator that supports CSI.

Before CSI, storage drivers were part of the Kubernetes core (called **in-tree plugins**). CSI moved them **out-of-tree**, making them independently versioned and maintained.

### 🔄 How CSI Works

```
PVC Created  ──▶  Kubernetes  ──▶  CSI Controller Plugin  ──▶  Storage Provider (EBS, EFS, etc.)
                                         │
Pod Scheduled  ──▶  kubelet  ──▶  CSI Node Plugin  ──▶  Mount volume to Pod
```

CSI has two main components:
- **Controller Plugin** – Handles volume creation, deletion, attach/detach
- **Node Plugin** – Handles mounting/unmounting volumes on the node

### ✅ Popular CSI Drivers

| Driver | Storage Type | Platform |
| ---------------------- | -------------------- | -------------- |
| **aws-ebs-csi-driver** | Block (EBS) | AWS |
| **aws-efs-csi-driver** | File (EFS/NFS) | AWS |
| **gce-pd-csi-driver** | Block (Persistent Disk) | GCP |
| **azuredisk-csi-driver**| Block (Managed Disk) | Azure |
| **nfs-subdir-external-provisioner** | File (NFS) | Any |
| **longhorn** | Distributed Block | Any |

### 📌 Key CSI Concepts

- **StorageClass** – Defines the type of storage and provisioner to use
- **PersistentVolume (PV)** – A piece of storage provisioned by the CSI driver
- **PersistentVolumeClaim (PVC)** – A request for storage by a Pod
- **VolumeSnapshot** – A point-in-time copy of a volume
- **Dynamic Provisioning** – Automatic creation of PVs when a PVC is created
- **Volume Expansion** – Ability to resize a PVC without downtime

---

## 🔄 CRI vs CNI vs CSI – Quick Comparison

| Aspect | CRI | CNI | CSI |
| -------------- | ----------------------- | ---------------------- | --------------------- |
| **Purpose** | Container lifecycle | Pod networking | Storage management |
| **Called by** | kubelet | Container runtime | Kubernetes controller |
| **Scope** | Node-level | Cluster-wide | Cluster-wide |
| **Examples** | containerd, CRI-O | Calico, Flannel, Cilium| EBS CSI, EFS CSI |
| **Config location** | `/etc/containerd/` | `/etc/cni/net.d/` | StorageClass YAML |

---

## 🧪 Useful Commands Reference

```bash
# CRI – Check container runtime
kubectl get nodes -o wide
crictl info
crictl ps

# CNI – Check networking
kubectl get pods -n kube-system          # Look for CNI pods (calico, flannel, etc.)
cat /etc/cni/net.d/*.conf               # CNI configuration on the node

# CSI – Check storage
kubectl get sc                           # List StorageClasses
kubectl get pv                           # List PersistentVolumes
kubectl get pvc --all-namespaces         # List PersistentVolumeClaims
kubectl get csidrivers                   # List installed CSI drivers

# Network Policy
kubectl get networkpolicy --all-namespaces
kubectl describe networkpolicy <POLICY_NAME> -n <NAMESPACE>

# Pod Security Admission
kubectl get ns <NAMESPACE> --show-labels
kubectl label ns <NAMESPACE> pod-security.kubernetes.io/enforce=restricted

# Cluster Upgrade & Node Management
kubectl get nodes                        # Check node versions
kubectl cordon <NODE>                    # Mark node unschedulable
kubectl drain <NODE> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <NODE>                  # Mark node schedulable again
kubectl get pdb --all-namespaces         # List Pod Disruption Budgets
```

---

## 🔒 Network Policy – Hands-On Lab

Network Policies allow you to control **Pod-to-Pod communication** at the network level. By default, all Pods in Kubernetes can communicate with each other. Network Policies let you **restrict** this behavior.

> ⚠️ Network Policies require a CNI plugin that supports them (e.g., **Calico**, **Cilium**). Flannel does **NOT** support Network Policies.

### 🔧 Step 1 – Setup the Environment

```bash
# Create namespaces
kubectl create ns client
kubectl create ns server

# Deploy a busybox pod in the client namespace
kubectl run busybox --image=busybox -n client -- sleep 3600

# Deploy an nginx pod in the server namespace
kubectl run nginx --image=nginx -n server

# Wait for pods to be ready
kubectl get pod -n client
kubectl get pod -n server
```

### 🔧 Step 2 – Install Calico (Network Policy Support)

```bash
# Install Calico policy-only manifest (for clusters that already have a CNI for networking)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico-policy-only.yaml

# Verify Calico pods are running
kubectl get pods -n kube-system | grep calico
```

### 🔧 Step 3 – Get the Server Pod IP

```bash
# Get the nginx pod IP in the server namespace
kubectl get pod -n server -o wide
```

Note the **IP address** of the nginx pod (e.g., `10.244.1.5`). You will use this IP to test connectivity.

### ✅ Step 4 – Test Pod-to-Pod Connection BEFORE Network Policy

```bash
# From the busybox pod in client namespace, curl the nginx pod IP
kubectl exec -n client busybox -- wget --spider --timeout=3 <NGINX_POD_IP>
```

**Expected Result:** ✅ Connection **succeeds** – busybox can reach nginx because there is no Network Policy restricting traffic.

---

### 🔧 Step 5 – Apply Default Deny All NetworkPolicy

This policy **blocks all incoming traffic** to all pods in the `server` namespace.

```yaml
# deny-all.yaml – Default Deny All Ingress in server namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: server
spec:
  podSelector: {}       # Applies to ALL pods in the namespace
  policyTypes:
  - Ingress              # Block all incoming traffic
```

```bash
# Apply the deny-all policy
kubectl apply -f deny-all.yaml

# Verify the policy
kubectl get networkpolicy -n server
```

### ❌ Step 6 – Test Pod-to-Pod Connection AFTER Deny All Policy

```bash
# Try to reach nginx again from the client namespace
kubectl exec -n client busybox -- wget --spider --timeout=3 <NGINX_POD_IP>
```

**Expected Result:** ❌ Connection **fails** / **times out** – all ingress traffic to the `server` namespace is now blocked by the deny-all policy.

---

### 🔧 Step 7 – Allow Traffic from Client Namespace

This policy **allows ingress traffic** to nginx only from pods in the `client` namespace.

```yaml
# allow-from-client.yaml – Allow traffic from client namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-client
  namespace: server
spec:
  podSelector:
    matchLabels:
      run: nginx           # Applies to the nginx pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: client    # Allow from client namespace
    ports:
    - protocol: TCP
      port: 80
```

```bash
# Apply the allow policy
kubectl apply -f allow-from-client.yaml

# Verify both policies are in place
kubectl get networkpolicy -n server
```

### ✅ Step 8 – Test Pod-to-Pod Connection AFTER Allow Policy

```bash
# Try to reach nginx again from the client namespace
kubectl exec -n client busybox -- wget --spider --timeout=3 <NGINX_POD_IP>
```

**Expected Result:** ✅ Connection **succeeds** – the `allow-from-client` policy explicitly permits traffic from the `client` namespace to the nginx pod on port 80.

### 🧹 Cleanup

```bash
kubectl delete ns client
kubectl delete ns server
```

---

## 🛡️ Pod Security Admission (PSA)

### 🔨 What is Pod Security Admission?

Pod Security Admission is a **built-in Kubernetes admission controller** that enforces Pod Security Standards at the namespace level. It replaced the deprecated PodSecurityPolicy (PSP) starting from Kubernetes v1.25.

PSA defines **three security levels**:

| Level | Description |
| -------------- | ------------------------------------------------------------- |
| **privileged** | No restrictions – allows everything (default) |
| **baseline** | Prevents known privilege escalations (e.g., hostNetwork, hostPID) |
| **restricted** | Heavily restricted – follows security best practices |

### 🔄 PSA Modes

Each security level can be applied in one of three **modes**:

| Mode | Behavior |
| ----------- | ------------------------------------------------------- |
| **enforce** | Pods that violate the policy are **rejected** |
| **audit** | Violations are recorded in the **audit log** but allowed |
| **warn** | Violations show a **warning** to the user but are allowed |

### 🔧 Step 1 – Create a Namespace with `enforce=restricted`

```bash
# Create a namespace with restricted Pod Security enforcement
kubectl create ns restricted-ns

# Label the namespace to enforce the restricted security standard
kubectl label ns restricted-ns \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version=latest
```

```bash
# Verify the labels
kubectl get ns restricted-ns --show-labels
```

### ❌ Step 2 – Create a Pod that Violates the Policy

The following pod runs as **root** and requests **privilege escalation**, which violates the `restricted` standard.

```yaml
# violation-pod.yaml – This pod WILL BE REJECTED
apiVersion: v1
kind: Pod
metadata:
  name: violation-pod
  namespace: restricted-ns
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      runAsUser: 0               # Running as root – VIOLATION
      allowPrivilegeEscalation: true   # Privilege escalation – VIOLATION
```

```bash
# Try to create the violating pod
kubectl apply -f violation-pod.yaml
```

**Expected Result:** ❌ The pod is **rejected** with an error message like:

```
Error from server (Forbidden): error when creating "violation-pod.yaml":
pods "violation-pod" is forbidden: violates PodSecurity "restricted:latest":
allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false),
runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true),
seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

### ✅ Step 3 – Create a Pod that Complies with the Policy

```yaml
# compliant-pod.yaml – This pod WILL BE ACCEPTED
apiVersion: v1
kind: Pod
metadata:
  name: compliant-pod
  namespace: restricted-ns
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - ALL
      seccompProfile:
        type: RuntimeDefault
```

```bash
# Create the compliant pod
kubectl apply -f compliant-pod.yaml

# Verify it is running
kubectl get pod -n restricted-ns
```

**Expected Result:** ✅ The pod is **created successfully** because it follows the `restricted` security standard.

### 🧹 Cleanup

```bash
kubectl delete ns restricted-ns
```

---

## 🔄 Kubernetes Cluster Upgrade – EKS 1.34 to 1.35

### 🔨 Why Upgrade?

Upgrading your Kubernetes cluster ensures you get:
- 🔐 Security patches and vulnerability fixes
- ✨ New features and API improvements
- 📦 Updated and supported add-on versions
- 🛡️ Continued vendor support (AWS EKS supports only the last 4 minor versions)

> ⚠️ **Important:** Kubernetes supports upgrading **one minor version at a time** (e.g., 1.34 → 1.35). You cannot skip versions.

---

### 📋 Step-by-Step Upgrade Process

### 1️⃣ Backup Everything

Before any upgrade, take backups of critical resources.

```bash
# Backup all cluster resources
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml

# Backup etcd (for self-managed clusters)
# For EKS, AWS manages etcd backups automatically

# Backup specific critical resources
kubectl get configmaps --all-namespaces -o yaml > configmaps-backup.yaml
kubectl get secrets --all-namespaces -o yaml > secrets-backup.yaml
kubectl get pv -o yaml > pv-backup.yaml
kubectl get pvc --all-namespaces -o yaml > pvc-backup.yaml
```

### 2️⃣ Read the Release Notes

Always read the official release notes before upgrading.

```
📖 Kubernetes Changelog  : https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/
📖 EKS Release Notes     : https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html
```

**Check for:**
- ❌ Deprecated or removed APIs
- ⚠️ Breaking changes in workload behavior
- 📦 Required add-on version updates

```bash
# Check if your manifests use deprecated APIs
kubectl get apiservices
kubectl api-resources
```

### 3️⃣ Test the Upgrade in Staging First

> 🚨 **Never upgrade production directly.** Always test in a staging/dev cluster first.

```bash
# Create a staging cluster matching production
eksctl create cluster --name staging-cluster --version 1.35 --region ap-southeast-1

# Deploy your workloads to staging and run tests
# Validate application behavior, health checks, and integrations
```

### 4️⃣ Setup Pod Disruption Budgets (PDB)

A **PodDisruptionBudget** ensures that a minimum number of pods remain available during voluntary disruptions like node drains, upgrades, and scaling events.

#### 🔨 What is a PDB?

| Field | Description |
| ------------------- | ---------------------------------------------------- |
| `minAvailable` | Minimum number of pods that must remain running |
| `maxUnavailable` | Maximum number of pods that can be unavailable |
| `selector` | Which pods this PDB applies to |

> ⚠️ Use **either** `minAvailable` **or** `maxUnavailable`, not both.

#### 🔧 Example YAML – PodDisruptionBudget

```yaml
# pdb.yaml – Ensure at least 1 replica is always available during disruptions
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: nginx
---
# Deployment that the PDB protects
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

```bash
# Apply the PDB and Deployment
kubectl apply -f pdb.yaml

# Verify PDB
kubectl get pdb
kubectl describe pdb nginx-pdb
```

#### 🔧 Example – PDB with maxUnavailable

```yaml
# pdb-max-unavailable.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb-max
spec:
  maxUnavailable: 1        # At most 1 pod can be down at a time
  selector:
    matchLabels:
      app: nginx
```

### 5️⃣ Update the Control Plane

For **EKS**, update the control plane version using the AWS CLI or `eksctl`.

```bash
# Check current cluster version
eksctl get cluster --name my-cluster --region ap-southeast-1

# Upgrade EKS control plane to 1.35
eksctl upgrade cluster --name my-cluster --version 1.35 --region ap-southeast-1 --approve

# Wait for the upgrade to complete and verify
eksctl get cluster --name my-cluster --region ap-southeast-1
kubectl version --short
```

> ⏱️ Control plane upgrade takes approximately 20-30 minutes. During this time, the **API server** may have brief downtime but **running workloads are not affected**.

### 6️⃣ Create New Worker Nodes Compatible with 1.35

Create a **new node group** running Kubernetes 1.35, then migrate workloads from old nodes.

```bash
# Create a new managed node group with version 1.35
eksctl create nodegroup \
  --cluster my-cluster \
  --name ng-1-35 \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 5 \
  --region ap-southeast-1

# Verify new nodes are Ready
kubectl get nodes
```

### 7️⃣ Cordon, Drain Old Nodes & Reschedule Pods to New Nodes

#### 📌 Understanding Cordon, Drain, and Uncordon

| Command | What it Does |
| ---------------------- | ------------------------------------------------------------- |
| `kubectl cordon` | Marks a node as **unschedulable** – no new pods will be placed on it |
| `kubectl drain` | **Evicts all pods** from a node (respects PDBs) and cordons it |
| `kubectl uncordon` | Marks a node as **schedulable** again |

```bash
# List all nodes and identify old 1.34 nodes
kubectl get nodes

# Step 1: Cordon the old node (prevent new pods from being scheduled)
kubectl cordon <OLD_NODE_NAME>

# Verify the node shows SchedulingDisabled
kubectl get nodes

# Step 2: Drain the old node (evict all pods, respects PDBs)
kubectl drain <OLD_NODE_NAME> \
  --ignore-daemonsets \
  --delete-emptydir-data \
  --grace-period=60

# Step 3: Verify pods are rescheduled to new nodes
kubectl get pods -o wide --all-namespaces

# Step 4: (If needed) Uncordon a node to make it schedulable again
kubectl uncordon <NODE_NAME>
```

> ⚠️ `kubectl drain` will **fail** if evicting a pod would violate its PDB. This is why Step 4 (setting up PDBs) is critical before draining.

```bash
# Repeat for each old node
kubectl drain <OLD_NODE_2> --ignore-daemonsets --delete-emptydir-data
kubectl drain <OLD_NODE_3> --ignore-daemonsets --delete-emptydir-data

# Once all pods are moved, delete the old node group
eksctl delete nodegroup \
  --cluster my-cluster \
  --name ng-1-34 \
  --region ap-southeast-1
```

### 8️⃣ Upgrade the Add-On Versions

After the control plane and nodes are upgraded, update the cluster add-ons to versions compatible with 1.35.

```bash
# List current add-ons
eksctl get addons --cluster my-cluster --region ap-southeast-1

# Update CoreDNS
eksctl update addon --name coredns --cluster my-cluster --region ap-southeast-1 --force

# Update kube-proxy
eksctl update addon --name kube-proxy --cluster my-cluster --region ap-southeast-1 --force

# Update VPC CNI
eksctl update addon --name vpc-cni --cluster my-cluster --region ap-southeast-1 --force

# Update EBS CSI Driver (if installed)
eksctl update addon --name aws-ebs-csi-driver --cluster my-cluster --region ap-southeast-1 --force

# Verify all add-ons are updated
eksctl get addons --cluster my-cluster --region ap-southeast-1
```

### ✅ Post-Upgrade Verification

```bash
# Verify cluster version
kubectl version --short

# Verify all nodes are running 1.35
kubectl get nodes

# Verify all system pods are healthy
kubectl get pods -n kube-system

# Verify all workloads are running
kubectl get pods --all-namespaces

# Check for any issues
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -20
```

### 📋 Upgrade Checklist Summary

| Step | Action | Status |
| ---- | ----------------------------------------------- | ------ |
| 1 | Backup all resources | ⬜ |
| 2 | Read release notes & check deprecated APIs | ⬜ |
| 3 | Test upgrade in staging cluster | ⬜ |
| 4 | Setup Pod Disruption Budgets | ⬜ |
| 5 | Upgrade EKS Control Plane to 1.35 | ⬜ |
| 6 | Create new worker nodes (1.35 node group) | ⬜ |
| 7 | Cordon & Drain old nodes, reschedule pods | ⬜ |
| 8 | Upgrade add-on versions (CoreDNS, kube-proxy, VPC CNI) | ⬜ |

---

# 🎓 Kubernetes Part 11 – CKA Certification Guide (Final Chapter)


## 📚 What You Will Learn

This is the **final chapter** of our Kubernetes learning series. Now that you have hands-on experience with Kubernetes on **AWS EKS** (Parts 1–10), it's time to get **officially certified**. The **Certified Kubernetes Administrator (CKA)** exam validates your ability to manage production-grade Kubernetes clusters.

✅ Understand the CKA exam format, platform, and rules
✅ Know the complete CKA syllabus and domain weights
✅ Map what you've already learned (Parts 1–10) to the CKA syllabus
✅ Learn YAML tips, tricks, and shortcuts — **you do NOT need to memorize YAML**
✅ Generate YAML quickly using `kubectl` imperative commands during the exam
✅ Get a curated list of reference sites, practice platforms, and documentation
✅ Understand how EKS experience translates to the CKA exam environment
✅ Practice with real exam-style scenario questions

---

## 🏛️ What is the CKA Certification?

The **Certified Kubernetes Administrator (CKA)** is a globally recognized certification offered by the **Cloud Native Computing Foundation (CNCF)** in partnership with the **Linux Foundation**.

| Detail | Info |
| --------------------- | ------------------------------------------------------------- |
| **Full Name** | Certified Kubernetes Administrator |
| **Provider** | CNCF / Linux Foundation |
| **Exam Type** | Performance-based (hands-on, NOT multiple choice) |
| **Duration** | 2 hours |
| **Passing Score** | 66% |
| **Number of Questions**| 15–20 hands-on tasks |
| **Exam Platform** | PSI Secure Browser (remote proctored) |
| **Cost** | $395 USD (includes one free retake) |
| **Validity** | 2 years from the date of certification |
| **Kubernetes Version** | The exam always uses a recent stable version (check CNCF site) |
| **OS in Exam** | Ubuntu Linux (terminal-based) |

---

## 🖥️ Exam Platform & Environment – How the Exam Works

### 🔨 Exam Platform: PSI Secure Browser

The CKA exam is conducted online through the **PSI Secure Browser**. You take it from your own computer — there is **no test center**.

#### 📋 Before the Exam

| Requirement | Detail |
| ----------------------------- | ------------------------------------------------- |
| **Valid Government ID** | Passport or national ID card |
| **Webcam + Microphone** | Built-in or external — must be working |
| **Stable Internet** | Minimum 1 Mbps upload/download |
| **Clean Desk Policy** | No books, notes, phones, or secondary monitors |
| **PSI Secure Browser** | Must be installed before exam day |
| **System Check** | Run the PSI compatibility check 24 hours before |

#### 💻 During the Exam

| Feature | Detail |
| ----------------------------- | -------------------------------------------------- |
| **Environment** | Linux terminal (Ubuntu) with `kubectl` pre-installed |
| **Multiple Clusters** | You will be asked to switch between clusters using `kubectl config use-context` |
| **Root Access** | You have `sudo` access on the nodes |
| **Copy-Paste** | Allowed within the exam terminal |
| **Notepad** | A built-in notepad is available in the exam interface |
| **Browser Tab** | **ONE extra tab** is allowed for Kubernetes documentation |

### ⚠️ EKS vs CKA Exam Environment – Key Difference

Since you studied Kubernetes on **AWS EKS**, here's what changes in the CKA exam:

| Feature | AWS EKS (What You Learned) | CKA Exam (What You'll Face) |
| --------------------- | -------------------------------------- | ----------------------------------------- |
| **Cluster Setup** | Managed by AWS (`eksctl`) | kubeadm-based vanilla clusters |
| **Control Plane** | AWS manages it (hidden from you) | You manage etcd, API server, scheduler |
| **Networking** | AWS VPC CNI plugin | Various CNI plugins (Calico, Flannel) |
| **Storage** | EBS/EFS CSI drivers | hostPath, emptyDir, NFS, PV/PVC |
| **Node Management** | Managed Node Groups | Manual `kubeadm join`, `kubectl drain` |
| **Upgrades** | `eksctl upgrade cluster` | `kubeadm upgrade` (manual process) |
| **RBAC** | IAM + Kubernetes RBAC | Pure Kubernetes RBAC (no IAM) |
| **etcd** | Hidden, managed by AWS | You must backup/restore etcd manually |

> 💡 **Good News:** All the core Kubernetes concepts you learned (Pods, Deployments, Services, ConfigMaps, Secrets, PV/PVC, Network Policies, RBAC, etc.) are **100% the same** in both EKS and the exam. Only the cluster management layer is different.

---

## 📋 CKA Exam Syllabus & Domain Weights

The CKA exam covers **5 domains**. Each domain has a specific weight in the overall score.

### 📊 Domain Breakdown

| Domain | Weight | Topics |
| ---- | ------------------------------------------ | ------ |
| 1 | **Cluster Architecture, Installation & Configuration** | 25% |
| 2 | **Workloads & Scheduling** | 15% |
| 3 | **Services & Networking** | 20% |
| 4 | **Storage** | 10% |
| 5 | **Troubleshooting** | 30% |

> 🎯 **Troubleshooting (30%)** is the highest weighted domain. Practice debugging broken clusters, pods, and services extensively.

---

### 1️⃣ Cluster Architecture, Installation & Configuration (25%)

| Topic | What to Know |
| ----------------------------------- | ------------------------------------------------------ |
| RBAC (Role, ClusterRole, Bindings) | Create Roles, bind to users/service accounts |
| kubeadm cluster setup | `kubeadm init`, `kubeadm join`, `kubeadm upgrade` |
| etcd backup & restore | `etcdctl snapshot save/restore` |
| Manage certificates | Know where certs live (`/etc/kubernetes/pki/`) |
| Cluster upgrade (kubeadm) | Upgrade control plane + worker nodes step-by-step |
| Network policies | Allow/deny traffic between pods (you practiced this!) |

**What you already know from this course:**
- ✅ RBAC basics (Part 5)
- ✅ Network Policies (Part 10)
- ✅ Cluster upgrade concepts (Part 10)

**What to additionally study:**
- ❗ `kubeadm init / join / upgrade` (EKS hides this from you)
- ❗ etcd backup and restore with `etcdctl`
- ❗ TLS certificate management

---

### 2️⃣ Workloads & Scheduling (15%)

| Topic | What to Know |
| ----------------------------------- | ------------------------------------------------------ |
| Deployments, ReplicaSets, DaemonSets | Create, scale, update, rollback |
| Jobs & CronJobs | One-time and scheduled tasks |
| ConfigMaps & Secrets | Mount as volumes or inject as env vars |
| Resource requests & limits | CPU/memory requests and limits |
| Static Pods | Create pods managed directly by kubelet |
| Node selectors & affinity | Schedule pods to specific nodes |
| Taints & tolerations | Prevent/allow pods on specific nodes |
| Pod Disruption Budgets | minAvailable, maxUnavailable |

**What you already know from this course:**
- ✅ Deployments, ReplicaSets, DaemonSets (Parts 3–6)
- ✅ Jobs & CronJobs (Part 9)
- ✅ ConfigMaps & Secrets (Part 5)
- ✅ Resource requests & limits (Part 9)
- ✅ Taints & tolerations (Part 6)
- ✅ Pod Disruption Budgets (Part 10)

**What to additionally study:**
- ❗ Static Pods (`/etc/kubernetes/manifests/`)
- ❗ Node affinity/anti-affinity rules

---

### 3️⃣ Services & Networking (20%)

| Topic | What to Know |
| ----------------------------------- | ------------------------------------------------------ |
| ClusterIP, NodePort, LoadBalancer | Service types and when to use each |
| Ingress & Ingress Controllers | HTTP routing, path-based routing |
| CoreDNS | How DNS works inside a cluster |
| Network Policies | Ingress/egress rules between pods and namespaces |
| CNI plugin basics | Understand what CNI does (don't need to install) |

**What you already know from this course:**
- ✅ Services – ClusterIP, NodePort, LoadBalancer (Parts 4–5)
- ✅ Ingress with ALB (Part 8)
- ✅ Network Policies (Part 10)
- ✅ CNI concepts (Part 10)
- ✅ ExternalName services (Part 9)

**What to additionally study:**
- ❗ CoreDNS troubleshooting
- ❗ Ingress with NGINX Ingress Controller (not ALB)

---

### 4️⃣ Storage (10%)

| Topic | What to Know |
| ----------------------------------- | ------------------------------------------------------ |
| PersistentVolumes (PV) | Create PV with hostPath, NFS |
| PersistentVolumeClaims (PVC) | Bind PVC to PV, use in Pods |
| StorageClasses | Dynamic provisioning |
| Volume types | emptyDir, hostPath, configMap, secret |
| Volume expansion | Resize PVC |

**What you already know from this course:**
- ✅ PV/PVC concepts (Parts 5–6)
- ✅ CSI and StorageClass concepts (Part 10)
- ✅ StatefulSet with volumes (Part 9)

**What to additionally study:**
- ❗ `hostPath` volumes (used frequently in the exam instead of EBS/EFS)
- ❗ Manual PV creation (no dynamic provisioning in some questions)

---

### 5️⃣ Troubleshooting (30%) ⭐ Most Important

| Topic | What to Know |
| ----------------------------------- | ------------------------------------------------------ |
| Application failures | Debug CrashLoopBackOff, ImagePullBackOff, pending pods |
| Cluster component failures | Fix broken kubelet, API server, scheduler, etcd |
| Networking issues | Debug service connectivity, DNS resolution |
| Node troubleshooting | Fix NotReady nodes, check kubelet logs |
| Log collection | `kubectl logs`, `kubectl describe`, `journalctl` |

**Key debugging commands you MUST master:**

```bash
# Pod debugging
kubectl describe pod <POD_NAME>
kubectl logs <POD_NAME>
kubectl logs <POD_NAME> -c <CONTAINER_NAME>    # Multi-container pods
kubectl logs <POD_NAME> --previous              # Logs from crashed container
kubectl exec -it <POD_NAME> -- /bin/sh

# Node debugging
kubectl describe node <NODE_NAME>
ssh <NODE_IP>
sudo systemctl status kubelet
sudo journalctl -u kubelet -f

# Service/Networking debugging
kubectl get endpoints <SERVICE_NAME>
kubectl run tmp --image=busybox --rm -it -- wget -qO- <SERVICE_NAME>:<PORT>
kubectl exec -it <POD> -- nslookup <SERVICE_NAME>

# Cluster component debugging
kubectl get componentstatuses              # Deprecated but may appear
kubectl get pods -n kube-system
sudo cat /var/log/containers/*.log
```

---

## ⌨️ YAML Tips & Tricks – You Do NOT Need to Memorize YAML

> 🚨 **This is the most important section for CKA exam preparation.** You do NOT need to memorize YAML manifests. The exam allows access to the official Kubernetes documentation, and `kubectl` has powerful commands to generate YAML for you.

### 🔧 Method 1 – Generate YAML with `kubectl` Imperative Commands (`--dry-run=client -o yaml`)

This is the **fastest way** to create YAML during the exam. Use `kubectl run` or `kubectl create` with the `--dry-run=client -o yaml` flags to generate a YAML template, then edit it.

```bash
# Generate a Pod YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Generate a Deployment YAML
kubectl create deployment nginx-dep --image=nginx --replicas=3 --dry-run=client -o yaml > deployment.yaml

# Generate a Service YAML (ClusterIP)
kubectl expose pod nginx --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml

# Generate a NodePort Service YAML
kubectl expose pod nginx --port=80 --type=NodePort --dry-run=client -o yaml > nodeport-svc.yaml

# Generate a Job YAML
kubectl create job my-job --image=busybox --dry-run=client -o yaml -- echo "Hello" > job.yaml

# Generate a CronJob YAML
kubectl create cronjob my-cron --image=busybox --schedule="*/5 * * * *" --dry-run=client -o yaml -- echo "Hello" > cronjob.yaml

# Generate a ConfigMap YAML
kubectl create configmap my-config --from-literal=key1=value1 --dry-run=client -o yaml > configmap.yaml

# Generate a Secret YAML
kubectl create secret generic my-secret --from-literal=password=abc123 --dry-run=client -o yaml > secret.yaml

# Generate a ServiceAccount YAML
kubectl create serviceaccount my-sa --dry-run=client -o yaml > sa.yaml

# Generate a Namespace YAML
kubectl create namespace my-ns --dry-run=client -o yaml > ns.yaml

# Generate a Role YAML
kubectl create role my-role --verb=get,list,watch --resource=pods --dry-run=client -o yaml > role.yaml

# Generate a RoleBinding YAML
kubectl create rolebinding my-binding --role=my-role --user=john --dry-run=client -o yaml > rolebinding.yaml

# Generate a ClusterRole YAML
kubectl create clusterrole my-clusterrole --verb=get,list --resource=nodes --dry-run=client -o yaml > clusterrole.yaml

# Generate a ClusterRoleBinding YAML
kubectl create clusterrolebinding my-binding --clusterrole=my-clusterrole --user=admin --dry-run=client -o yaml > crb.yaml

# Generate a NetworkPolicy YAML (start from docs, no imperative shortcut)

# Generate a PodDisruptionBudget YAML
kubectl create pdb my-pdb --min-available=1 --selector=app=nginx --dry-run=client -o yaml > pdb.yaml

# Generate an Ingress YAML
kubectl create ingress my-ingress --rule="host.com/path=svc:80" --dry-run=client -o yaml > ingress.yaml
```

### 🔧 Method 2 – Use `kubectl explain` to Look Up YAML Fields

During the exam, if you forget a field name or the structure of a resource, use `kubectl explain`.

```bash
# Top-level fields of a Pod
kubectl explain pod.spec

# Nested fields – containers
kubectl explain pod.spec.containers

# Volume mounts
kubectl explain pod.spec.containers.volumeMounts

# Deployment strategy
kubectl explain deployment.spec.strategy

# PersistentVolumeClaim spec
kubectl explain pvc.spec

# NetworkPolicy spec
kubectl explain networkpolicy.spec

# Get the FULL structure recursively
kubectl explain pod --recursive | head -80

# Search for a specific field
kubectl explain pod --recursive | grep -i "tolerations" -A 5
```

### 🔧 Method 3 – Copy YAML from the Official Kubernetes Documentation

During the exam, you are allowed **ONE browser tab** open to the official docs:

```
✅ Allowed: https://kubernetes.io/docs/
✅ Allowed: https://kubernetes.io/blog/
✅ Allowed: https://helm.sh/docs/
```

**Pro tip:** Use the **search bar** on kubernetes.io/docs to find YAML examples quickly. Almost every resource type has a ready-to-copy YAML example.

**Fastest pages to bookmark:**

| Resource | Documentation Page |
| ---------------------- | ------------------------------------------------------------------ |
| Pod | `kubernetes.io/docs/concepts/workloads/pods/` |
| Deployment | `kubernetes.io/docs/concepts/workloads/controllers/deployment/` |
| Services | `kubernetes.io/docs/concepts/services-networking/service/` |
| Ingress | `kubernetes.io/docs/concepts/services-networking/ingress/` |
| PV & PVC | `kubernetes.io/docs/concepts/storage/persistent-volumes/` |
| Network Policy | `kubernetes.io/docs/concepts/services-networking/network-policies/` |
| RBAC | `kubernetes.io/docs/reference/access-authn-authz/rbac/` |
| etcd backup | `kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/` |
| kubeadm upgrade | `kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/` |
| Taints & Tolerations | `kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/` |
| kubectl Cheat Sheet | `kubernetes.io/docs/reference/kubectl/cheatsheet/` |

### 🔧 Method 4 – Useful `kubectl` Shortcuts for Speed

Speed matters in the CKA exam. Set up these aliases at the **start of the exam**.

```bash
# Add these aliases at the start of the exam
alias k=kubectl
alias kn='kubectl config set-context --current --namespace'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgn='kubectl get nodes'
alias kd='kubectl describe'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'

# Enable auto-completion (usually pre-enabled in the exam)
source <(kubectl completion bash)
complete -o default -F __start_kubectl k

# Set default editor (vim is pre-installed)
export EDITOR=vim
```

#### ⌨️ Essential Vim Commands for the Exam

```
i          → Enter insert mode (start typing)
Esc        → Exit insert mode
:wq        → Save and quit
:q!        → Quit without saving
dd         → Delete a line
yy         → Copy a line
p          → Paste below current line
u          → Undo
/search    → Search for text
n          → Next search result
:set nu    → Show line numbers
:%s/old/new/g → Find and replace all
gg         → Go to top of file
G          → Go to bottom of file
```

#### 🔧 Quick YAML Editing Tips

```bash
# Quickly edit a running resource
kubectl edit deployment nginx-dep

# Patch a resource without editing the full YAML
kubectl patch deployment nginx-dep -p '{"spec":{"replicas":5}}'

# Scale without editing YAML
kubectl scale deployment nginx-dep --replicas=5

# Set image without editing YAML
kubectl set image deployment/nginx-dep nginx=nginx:1.25

# Add labels without YAML
kubectl label pod nginx app=web

# Add annotations without YAML
kubectl annotate pod nginx description="my pod"

# Taint a node without YAML
kubectl taint nodes node1 key=value:NoSchedule

# Remove a taint
kubectl taint nodes node1 key=value:NoSchedule-
```

---

## 📝 CKA Practice Scenarios

These scenarios mirror real CKA exam questions. Practice each one in a local cluster (minikube or kind).

### 📌 Scenario 1 – Create a Pod with Specific Requirements

> Create a pod named `web-pod` in the `production` namespace using the `nginx:1.25` image. The pod should have a CPU request of `250m`, a memory limit of `256Mi`, and a label `tier=frontend`.

**💡 How to approach:** Use `--dry-run=client -o yaml` to generate the base YAML, then edit only what's needed.

```bash
# Step 1: Create the namespace
kubectl create namespace production

# Step 2: Generate the Pod YAML using --dry-run (DO NOT memorize YAML!)
kubectl run web-pod -n production --image=nginx:1.25 \
  --labels=tier=frontend \
  --dry-run=client -o yaml > web-pod.yaml
```

This generates a complete Pod YAML. Now you only need to add the `resources` block.

```bash
# Step 3: If you forget the resources field structure, use kubectl explain
kubectl explain pod.spec.containers.resources
kubectl explain pod.spec.containers.resources.requests
```

```bash
# Step 4: Edit the generated YAML to add resources
vi web-pod.yaml
```

Add the following under `containers`:

```yaml
        resources:
          requests:
            cpu: "250m"
          limits:
            memory: "256Mi"
```

```bash
# Step 5: Apply and verify
kubectl apply -f web-pod.yaml
kubectl get pod web-pod -n production
kubectl describe pod web-pod -n production
```

---

### 📌 Scenario 2 – RBAC: Create a Role and Bind it to a User

> Create a Role named `pod-reader` in the `dev` namespace that allows `get`, `list`, and `watch` on Pods. Bind it to a user named `jane`.

**💡 How to approach:** RBAC resources can be created **entirely with imperative commands** – no YAML file needed at all!

```bash
# Step 1: Create the namespace
kubectl create namespace dev

# Step 2: Create the Role (imperative – no YAML needed!)
kubectl create role pod-reader -n dev --verb=get,list,watch --resource=pods

# Step 3: Create the RoleBinding (imperative – no YAML needed!)
kubectl create rolebinding pod-reader-binding -n dev --role=pod-reader --user=jane

# Step 4: Verify – can jane list pods?
kubectl auth can-i list pods -n dev --as=jane       # Should return: yes
kubectl auth can-i delete pods -n dev --as=jane     # Should return: no
```

**💡 If the question asks for YAML instead:**

```bash
# Generate Role YAML
kubectl create role pod-reader -n dev --verb=get,list,watch --resource=pods \
  --dry-run=client -o yaml > role.yaml

# Generate RoleBinding YAML
kubectl create rolebinding pod-reader-binding -n dev --role=pod-reader --user=jane \
  --dry-run=client -o yaml > rolebinding.yaml
```

---

### 📌 Scenario 3 – Expose a Deployment as a NodePort Service

> Create a deployment named `httpd-deploy` with 3 replicas using the `httpd:2.4` image. Expose it as a NodePort service on port 80.

**💡 How to approach:** Both the Deployment and Service can be created with imperative commands.

```bash
# Step 1: Create the deployment (imperative)
kubectl create deployment httpd-deploy --image=httpd:2.4 --replicas=3

# Step 2: Expose it as a NodePort service (imperative)
kubectl expose deployment httpd-deploy --type=NodePort --port=80

# Step 3: Verify
kubectl get deployment httpd-deploy
kubectl get svc httpd-deploy
kubectl get pods -l app=httpd-deploy
```

**💡 If you need to generate the YAML files instead:**

```bash
# Generate Deployment YAML
kubectl create deployment httpd-deploy --image=httpd:2.4 --replicas=3 \
  --dry-run=client -o yaml > deployment.yaml

# Generate Service YAML
kubectl expose deployment httpd-deploy --type=NodePort --port=80 \
  --dry-run=client -o yaml > service.yaml
```

---

### 📌 Scenario 4 – etcd Backup and Restore

> Take a snapshot of etcd and save it to `/opt/etcd-backup.db`. Then restore it.

> 🚨 **Do you need to memorize these long etcd commands? NO!**
> The etcd backup/restore commands are long and have many flags. You do **NOT** need to memorize them. Here's how to find them during the exam.

**💡 How to find the etcd commands during the exam:**

```
Step 1: Open your allowed browser tab → https://kubernetes.io/docs/
Step 2: Search for "etcd backup" in the search bar
Step 3: Go to → "Operating etcd clusters for Kubernetes"
        (kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
Step 4: Scroll to "Backing up an etcd cluster" → the exact commands are there!
```

**💡 How to find the certificate paths (you don't memorize these either):**

```bash
# The etcd pod manifest contains ALL the certificate paths you need!
# Just read it:
cat /etc/kubernetes/manifests/etcd.yaml

# Look for these flags in the etcd container command:
# --cert-file       → this is your --cert
# --key-file        → this is your --key
# --trusted-ca-file → this is your --cacert
# --listen-client-urls → this is your --endpoints
```

**Now run the backup using values from the manifest:**

```bash
# Backup etcd (copy the command from kubernetes.io docs, fill in cert paths from the manifest)
ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the snapshot
ETCDCTL_API=3 etcdctl snapshot status /opt/etcd-backup.db --write-table

# Restore etcd (to a new data directory)
ETCDCTL_API=3 etcdctl snapshot restore /opt/etcd-backup.db \
  --data-dir=/var/lib/etcd-restored
```

**After restore, update the etcd manifest:**

```bash
# Edit the etcd static pod manifest
vi /etc/kubernetes/manifests/etcd.yaml

# Change the hostPath volume from:
#   path: /var/lib/etcd
# To:
#   path: /var/lib/etcd-restored

# The kubelet will automatically restart etcd with the restored data
```

> 📝 **Summary:** Don't memorize → Read `/etc/kubernetes/manifests/etcd.yaml` for cert paths, copy the command template from kubernetes.io/docs, fill in the values.

---

### 📌 Scenario 5 – Troubleshoot a Broken Node

> A worker node `node01` is in `NotReady` state. Fix it.

**💡 How to approach:** Troubleshooting follows a fixed pattern. No YAML needed – it's all commands.

```bash
# Step 1: Check node status and look for the reason
kubectl describe node node01
# Look at "Conditions" section → what condition is False/Unknown?

# Step 2: SSH into the broken node
ssh node01

# Step 3: Check kubelet status (kubelet is the most common cause)
sudo systemctl status kubelet

# Step 4: If kubelet is stopped/dead, start and enable it
sudo systemctl start kubelet
sudo systemctl enable kubelet

# Step 5: If kubelet is running but erroring, check the logs
sudo journalctl -u kubelet -f
# Read the error messages carefully – they usually tell you exactly what's wrong

# Step 6: Common fixes based on error messages
# ERROR: "unable to load client CA file"
#   → Fix the CA cert path in /var/lib/kubelet/config.yaml

# ERROR: "failed to connect to apiserver"
#   → Check the API server address in kubelet config

# ERROR: kubelet just won't start
#   → sudo systemctl daemon-reload
#   → sudo systemctl restart kubelet
```

---

### 📌 Scenario 6 – Cluster Upgrade with kubeadm

> Upgrade the control plane node from Kubernetes 1.31.0 to 1.32.0 using kubeadm.

> 🚨 **Do you need to memorize the upgrade steps? NO!** Search for **"kubeadm upgrade"** in the Kubernetes docs.
> `kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/`

**💡 The docs give you the exact commands. Here's the flow:**

```bash
# Step 1: Drain the control plane node (stop scheduling + evict pods)
kubectl drain controlplane --ignore-daemonsets

# Step 2: Upgrade kubeadm to the target version
sudo apt-get update
sudo apt-get install -y kubeadm=1.32.0-*

# Step 3: Check what the upgrade will do
sudo kubeadm upgrade plan

# Step 4: Apply the upgrade
sudo kubeadm upgrade apply v1.32.0

# Step 5: Upgrade kubelet and kubectl
sudo apt-get install -y kubelet=1.32.0-* kubectl=1.32.0-*
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Step 6: Make the node schedulable again
kubectl uncordon controlplane

# Step 7: Verify
kubectl get nodes
```

---

### 📌 Scenario 7 – Network Policy

> In the `secure` namespace, create a NetworkPolicy that allows ingress traffic to pods with label `app=db` only from pods with label `app=api` on port 3306.

> 🚨 **NetworkPolicy has NO imperative shortcut.** You must write YAML for this. But you can **copy the template from the Kubernetes docs:**
> Search: `kubernetes.io/docs/concepts/services-networking/network-policies/`

**💡 How to approach:** Copy the example from the docs, then modify the labels, namespace, and ports.

```bash
# Step 1: Create the namespace
kubectl create namespace secure

# Step 2: Use kubectl explain to check the structure if needed
kubectl explain networkpolicy.spec
kubectl explain networkpolicy.spec.ingress
```

```bash
# Step 3: Create the YAML file (copy template from docs, modify it)
vi netpol.yaml
```

```yaml
# netpol.yaml – copied from docs, modified for our requirements
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: secure
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 3306
```

```bash
# Step 4: Apply and verify
kubectl apply -f netpol.yaml
kubectl get networkpolicy -n secure
kubectl describe networkpolicy allow-api-to-db -n secure
```

---

### 📌 Scenario 8 – Persistent Volume and Claim

> Create a PersistentVolume named `pv-log` of `1Gi` using `hostPath` at `/var/log/app`. Create a PVC named `pvc-log` that binds to it. Mount it in a pod.

> 🚨 **PV/PVC also have no full imperative shortcut.** Copy the YAML template from Kubernetes docs:
> Search: `kubernetes.io/docs/concepts/storage/persistent-volumes/`

**💡 How to approach:** Copy the PV and PVC examples from docs. For the Pod, generate it with `--dry-run` and add the volume mount.

```bash
# Step 1: Use kubectl explain to check fields if needed
kubectl explain pv.spec
kubectl explain pv.spec.hostPath
kubectl explain pvc.spec

# Step 2: Create the PV and PVC YAML (copy from docs, modify values)
vi pv-pvc.yaml
```

```yaml
# pv-pvc.yaml – PV and PVC copied from docs, modified for our requirements
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/log/app
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-log
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
# Step 3: Generate the Pod YAML with --dry-run, then add the volume
kubectl run log-pod --image=busybox \
  --dry-run=client -o yaml \
  -- sh -c "while true; do echo \$(date) >> /app-log/output.log; sleep 5; done" > pod.yaml

# Step 4: Edit pod.yaml to add volumeMounts and volumes
vi pod.yaml
```

Add the following to the container and pod spec:

```yaml
    volumeMounts:
    - mountPath: /app-log
      name: log-volume
  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: pvc-log
```

```bash
# Step 5: Apply everything and verify
kubectl apply -f pv-pvc.yaml
kubectl apply -f pod.yaml

kubectl get pv
kubectl get pvc
kubectl get pods
kubectl describe pvc pvc-log
```

---

## 🏋️ Where to Practice

### 🖥️ Practice Platforms

| Platform | What it Offers |
| ------------------------------ | -------------------------------------------------- |
| **Killer Shell (killer.sh)** | 2 free CKA simulator sessions (included with exam purchase) — most realistic CKA exam simulator |
| **Play with Kubernetes** | Free browser-based K8s cluster (limited time sessions) |
| **minikube (local)** | Run a local single-node cluster on your laptop |
| **kind (Kubernetes in Docker)** | Lightweight multi-node local cluster for quick practice |
| **kubeadm on VMs** | Set up a real multi-node cluster on EC2/VirtualBox — best for CKA practice |

> 💡 **Best practice strategy:** Set up a cluster using `kubeadm` on 2–3 VMs (1 control plane + 2 workers). This is exactly how the CKA exam environment works. minikube and EKS abstract away the cluster internals that CKA tests you on.

### 📖 Official Documentation & References (Allowed During Exam)

These are the **only websites allowed** during the CKA exam. Bookmark them and practice navigating them quickly.

| Resource | URL |
| --------------------------------- | -------------------------------------------------- |
| **Kubernetes Official Docs** ⭐ | `https://kubernetes.io/docs/` |
| **Kubernetes Blog** | `https://kubernetes.io/blog/` |
| **Helm Docs** | `https://helm.sh/docs/` |
| **kubectl Cheat Sheet** ⭐ | `https://kubernetes.io/docs/reference/kubectl/cheatsheet/` |
| **kubectl Reference** | `https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands` |

### 📖 Official CKA Exam Resources

| Resource | URL |
| --------------------------------- | -------------------------------------------------- |
| **CKA Exam Curriculum (Syllabus)**| `https://github.com/cncf/curriculum` |
| **CKA Registration Page** | `https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/` |
| **CNCF CKA FAQ** | `https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks` |
| **PSI Exam System Requirements** | `https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2/exam-preparation-checklist` |
| **Exam Candidate Handbook** | `https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2` |

### 📖 Kubernetes Core Documentation

| Resource | URL |
| --------------------------------- | -------------------------------------------------- |
| **Kubernetes GitHub** | `https://github.com/kubernetes/kubernetes` |
| **Kubernetes API Reference** | `https://kubernetes.io/docs/reference/kubernetes-api/` |
| **etcd Official Documentation** | `https://etcd.io/docs/` |
| **kubeadm Documentation** | `https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/` |
| **Kubernetes Release Notes** | `https://kubernetes.io/releases/` |

### 📖 Most Useful Doc Pages for the CKA Exam (Bookmark These!)

```
📌 Pods              → kubernetes.io/docs/concepts/workloads/pods/
📌 Deployments       → kubernetes.io/docs/concepts/workloads/controllers/deployment/
📌 Services          → kubernetes.io/docs/concepts/services-networking/service/
📌 Ingress           → kubernetes.io/docs/concepts/services-networking/ingress/
📌 PV & PVC          → kubernetes.io/docs/concepts/storage/persistent-volumes/
📌 Network Policy    → kubernetes.io/docs/concepts/services-networking/network-policies/
📌 RBAC              → kubernetes.io/docs/reference/access-authn-authz/rbac/
📌 etcd backup       → kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
📌 kubeadm upgrade   → kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
📌 Taints/Tolerations→ kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
📌 Static Pods       → kubernetes.io/docs/tasks/configure-pod-container/static-pod/
📌 kubectl Cheat Sheet→ kubernetes.io/docs/reference/kubectl/cheatsheet/
```

---

## 📊 Mapping This Course (Parts 1–10) to CKA Domains

| CKA Domain | This Course Coverage | What's Extra for CKA |
| ------- | ----------------------------------------- | ---- |
| **Cluster Setup (25%)** | Cluster creation (Part 1), RBAC (Part 5), Network Policy (Part 10), Upgrade (Part 10) | kubeadm, etcd backup/restore, certificates |
| **Workloads (15%)** | Pods, Deployments, ReplicaSets (Parts 2–6), Jobs/CronJobs (Part 9), PDB (Part 10) | Static Pods, Node affinity |
| **Services & Networking (20%)** | Services (Parts 4–5), Ingress/ALB (Part 8), CNI/Network Policy (Part 10) | CoreDNS debugging, NGINX Ingress |
| **Storage (10%)** | PV/PVC (Parts 5–6), CSI concepts (Part 10), StatefulSets (Part 9) | hostPath volumes, manual PV |
| **Troubleshooting (30%)** | Probes & Init Containers (Part 7), Pod debugging | kubelet logs, etcd issues, broken nodes |

> 🎯 You already cover approximately **60–70%** of the CKA syllabus from this course. Focus your additional study on **kubeadm**, **etcd**, and **troubleshooting**.

---

## 🗓️ Recommended Study Plan

| Week | Focus Area | Action |
| ---- | ---------------------------------------- | ------------------------------------------ |
| 1 | Review Parts 1–10 of this course | Redo all hands-on exercises from scratch |
| 2 | kubeadm cluster setup & etcd | Set up a cluster with kubeadm on VMs (EC2/VirtualBox), practice etcd backup/restore |
| 3 | RBAC, Storage, Troubleshooting | Deep dive into Roles/Bindings, PV/PVC with hostPath, debug broken pods/nodes |
| 4 | Practice with Killer Shell | Complete your 2 free Killer Shell simulator sessions (included with exam purchase) |
| 5 | Weak areas + Speed training | Time yourself — practice generating YAML with `--dry-run`, aim to finish 15 questions in 90 minutes |
| 6 | **Exam Day** | Take the exam with confidence! 🎉 |

---

## 🏆 Exam Day Tips

1. ⏱️ **Time Management** — You have 2 hours for ~17 questions. That's ~7 minutes per question. If stuck, flag it and move on.
2. 📋 **Read the question carefully** — Check the namespace, cluster context, and exact resource names.
3. ⌨️ **Set up aliases first** — Spend the first 2 minutes setting up `alias k=kubectl` and auto-completion.
4. 📖 **Use the docs** — Don't waste time memorizing. Copy YAML from kubernetes.io/docs.
5. 🔄 **Use `--dry-run=client -o yaml`** — Generate YAML imperatively, then edit. Never write YAML from scratch.
6. ✅ **Verify your work** — After every task, run `kubectl get` and `kubectl describe` to confirm.
7. 🔀 **Switch contexts** — Always run the `kubectl config use-context` command given at the top of each question.
8. 💾 **Don't forget to save** — In Vim, always `:wq` before moving on.
9. 🧘 **Stay calm** — 66% is the passing score. You can afford to miss a few questions.
10. 🔁 **Use your free retake** — If you don't pass, you get one free retake. Learn from the experience.

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

