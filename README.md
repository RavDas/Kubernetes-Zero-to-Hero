# Kubernetes-Zero-to-Hero
Creating this repo with an intent to make Kubernetes easy for begineers. This is a work-in-progress repo.

## Kubernetes Installation Using KOPS on EC2

### Create an EC2 instance or use your personal laptop.

Dependencies required 

1. Python3
2. AWS CLI
3. kubectl

###  Install dependencies

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

```
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```

```
sudo apt-get update
sudo apt-get install -y python3-pip apt-transport-https kubectl
```

```
pip3 install awscli --upgrade
```

```
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

### Install KOPS (our hero for today)

```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops
```

### Provide the below permissions to your IAM user. If you are using the admin user, the below permissions are available by default

1. AmazonEC2FullAccess
2. AmazonS3FullAccess
3. IAMFullAccess
4. AmazonVPCFullAccess

### Set up AWS CLI configuration on your EC2 Instance or Laptop.

Run `aws configure`

## Kubernetes Cluster Installation 

Please follow the steps carefully and read each command before executing.

### Create S3 bucket for storing the KOPS objects.

```
aws s3api create-bucket --bucket kops-abhi-storage --region us-east-1
```

### Create the cluster 

```
kops create cluster --name=demok8scluster.k8s.local --state=s3://kops-abhi-storage --zones=us-east-1a --node-count=1 --node-size=t2.micro --master-size=t2.micro  --master-volume-size=8 --node-volume-size=8
```
Here the domain is used as "demok8scluster.k8s.local" local domain. But in production we use domains like amazon.com,google.com,xyz.com etc. Then we have configure this domain on Route 53.

So we can configure using below.

```
aws route53 create-hosted-zone --name <dev.example.com> --caller-reference 1
```

### Important: Edit the configuration as there are multiple resources created which won't fall into the free tier.

```
kops edit cluster myfirstcluster.k8s.local
```

Step 12: Build the cluster

```
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-abhi-storage
```

This will take a few minutes to create............

After a few mins, run the below command to verify the cluster installation.

```
kops validate cluster demok8scluster.k8s.local
```


### Deploy an App using Kubernetes Pods

Install kubectl binary with curl on Windows 

```
curl.exe -LO "https://dl.k8s.io/release/v1.30.0/bin/windows/amd64/kubectl.exe"

```
Check kubectl installation

```
kubectl version
```

Here I am going to install a local Kubernetes cluster using minikube.

Install minikube.

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
Choose the command according to the OS and the architecture. Visit -> https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download

To start the Kubernetes cluster

```
minikube start

or

minikube start --memory=4096 --driver=hyperkit

```

In minikube, it just creates a Virtual Machine on on top of it a single node kubernetes cluster is created. But in production environement, there will be master nodes and worker nodes in a cluster.

To check nodes

```
kubectl get nodes
```

Creation of pods

```
vi simple-pod.yml
```

```
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
kubectl create -f simple-pod.yml

To view pods

```
kubectl get pods -o wide
```

Login into the kubernetes cluster

```
minikube ssh
```
And type

```
curl <ip address of the cluster>
```

In real-time kubernetes cluster, we have ssh into the master/ worker nodes using their ip addresses.

To get all information of the pod,

```
kubectl describe pod nginx
```

To debug/view the logs of the pods,

```
kubectl logs nginx
```
here nginx is the name of the pod i have earlier created.


To provide auto healing and auto scaling for the cluster, continue.......

