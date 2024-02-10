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

====================================================================================

### Additional note 

### Data Plane/ Worker Node
```
In java you have java runtime to run application
In docker you have docker engine and docker shimp to run the container
similary, in k8 to deploy and run the pods we have something called kubelet. they are found in worker node.
Its responsible for maintaining the pod, and inform k8 if not running, usually it informs the master node.

- Docker has only one runtime dockershimp but in case of k8 we have multiple support like container-d-, cri-o- etc.
 These all helps us to run container inside the pods.

-kubeproxy provides networking, ip-address and loadbalancing feature

so we have container-run time which will run pod, kubelet which will maintian our pod and inform control plane
or master node if something goes wrong and lastly kube-proxy for networking, scaling, ip-address and load balancing
feature.

These all are data-plane or worker node component
```
### Control Plane/Master Node

```
API sever - The heart of the k8 is API server which exposes k8 to the outer world.
The data plane/worker node is internal and helps to run the application. but this API server
which is in master node helps to take request from the outside world.

-scheduler - when user tries to create the pod then he/she will use the API server and to schedule the pod
creation in particular worker node there is anothe component called scheduler.

-ETCD - when we want to store the entire cluster information then in master node there is another component called etcd.
this component is used for the backup of the cluster, since all info of cluster is saved if something goes wrong on the cluster,
this info is used to spin another pod. its a key value store.

-controller manager => autoscaling, replicaset, insure the replicaset and autoscaling part is working properly. manager for inbuilt controller.

-cloud-control-manager -> suppose you want to create a load balancer or storage in the aws, eks then at that time when we give
command there has to be a underlying component that will help to understand those command and order aws to spin up those fetaure
```
### Manage Hundreds of Kubernetes clusters
```
- KOPS is widely used.
### Deploy app:
pod => how to run a container
- 1 or group of container => pod
- app deployed in container have some dependency or trying to read some config file
  then instead of creating a separate pod for each of them we create one pod and
  put both the container in same pod and this will have some benefit like shared
  networkign and storgae. this way container 1 can communicate with another pod with
  localhost. 

 -kubectl => command line for k8 which help to communicate with k8 cluster
 -kubectl get pods
 - kubectl describe pod my-app-pod

   four root level properties:
   apiVersion:
   kind:
   metadata:
   spec:
```
```
apiVersion: v1
kind: Pod
metadata:
 name: postgres
 labels: 
  tier: db-tier
spec:
 containers:
  -name: postgres
  image: postgres
```
```
controlplane ~ ✖ kubectl run nginx --image=nginx
```
### how to scale the replicaset in kubernetes
```
change in the file and then run this command
kubectl replace -f replicaset -defination.yml

kubectl scale --replicas=6 -f replicaset-defination.yml

kubectl scale --replicas=6 replicaset myapp-replicaset
```




