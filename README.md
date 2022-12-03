AWS Elastic Kubernetes Service (EKS) Kubeflow QuickStart  
=======================================================
Abstract:
```
With the Advent of integrated development environment (IDE) for machine learning (ML) that provide
a single visual interface where you can access purpose-built tools to perform all (ML) development 
steps, from preparing data to building, training, and deploying your ML models, improving data.

The Kubeflow project is dedicated to making deployments of machine learning (ML) workflows on
Kubernetes simple, portable and scalable.

This quickstart will show you how to provision an AWS EKS Cluster with Kubeflow installed.

```
This solution shows how to create an AWS EKS Cluster with Kubeflow support.  
```
Note: This how-to assumes you are creating the eks cluster in us-east-1, you have access to your AWS Root
Account, and you can login to an EC2 Instance remotely.
```
Steps:  
* [Create an EC2 Instance with cloud-init](#create-an-ec2-instance-with-cloud-init)   
* [Remove AWS EKS Cluster and Resources](#remove-aws-eks-cluster-and-resources)  

## Create an EC2 Instance with cloud-init
We'll using an EC2 instance to install awscli , eksctl, and other support programs needed so we can create the EKS Cluster,  
This is a step by step process.

### Create an EC2 Instance
#### AWS EC2 Dashboard
Using AWS Managment Console goto the AWS EC2 Dashboard  
Click on "Launch Instance"  

Name and tags
```
Name: kubeflow_cloud_shell
```
Application and OS Images (Amazon Machine Image)
```
Search: ubuntu-bionic-18.04-amd64-server-20220901 (Community AMIs tab)
```
Choose AMI  
```
Ubuntu Server 22.04 LTS (HVM), SSD Volume Type 
```  
Click on "Select"

Instance Type
```
t2.micro
```
Key pair (login)
```
Your Account Keypair
```
Network settings
You could use the default security group if you've already added ssh as an inbound rule
```
Select "Create Security Group"
Select "Allow SSH traffic from"
Select "Anywhere"
```
Review and Launch  

### Connect to EC2 Instance


### Set Kubeflow Versioning and Clone kubeflow Repo
This will checkout the Kubeflow project which contains scripts to help launch Kubeflow
```
export KUBEFLOW_RELEASE_VERSION=v1.6.1
export AWS_RELEASE_VERSION=main

git clone https://github.com/awslabs/kubeflow-manifests.git && cd kubeflow-manifests
git checkout ${AWS_RELEASE_VERSION}
git clone --branch ${KUBEFLOW_RELEASE_VERSION} https://github.com/kubeflow/manifests.git upstream
```

### Install Cloud Shell tools
make install-tools

alias python=python3.8
python
exit()

### Configure AWS CLI
Use the AWS CLI to set Access Key, Secret Key, and Region Name
```
aws configure --profile=kubeflow
AWS Access Key ID []: <Your Access Key ID>
AWS Secret Access Key []: <Your Secret Access Key>
Default region name []: us-east-1
```
Test awscli to insure it has access to AWS Resources
```
aws s3 ls
```

### Create Cluster (takes about 20 minutes)
Using eksctl create the cluster
```
export CLUSTER_NAME=kubeflow
export CLUSTER_REGION=us-east-1

eksctl create cluster \
--name ${CLUSTER_NAME} \
--version 1.22 \
--region ${CLUSTER_REGION} \
--nodegroup-name linux-nodes \
--node-type m5.xlarge \
--nodes 5 \
--nodes-min 5 \
--nodes-max 10 \
--managed \
--with-oidc
    
```
Wait till cluster creation has completed before proceeding.  

### Cluster status
Insure the cluster status is "Active"
```
aws eks describe-cluster --region us-east-1 --name kubeflow --query "cluster.status"
```
### Test Cluster Nodes
Use kubectl test status of cluster nodes
```
kubectl get nodes
```
### Deploy kubeflow
```
make deploy-kubeflow INSTALLATION_OPTION=helm DEPLOYMENT_OPTION=vanilla
```

# Check kubeflow status
```
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n auth
kubectl get pods -n knative-eventing
kubectl get pods -n knative-serving
kubectl get pods -n kubeflow
kubectl get pods -n kubeflow-user-example-com
```

### Connect to EC2 Instance redirecting port 8001 Locally
Using ssh from your local machine, open a tunnel to your AWS EC2 Instance
```
ssh -i <AWS EC2 Private Key> ec2-user@<AWS EC2 Instance IP Address> -L 8001:localhost:8001
```
### Connect to Dashboard using Local Browser
Using your local client-side browser enter the following URL. The configure-kube-dashboard script
also generated a "Security Token" required to login to the dashboard.
```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

## Remove AWS EKS Cluster and Resources
You will need to ssh into the AWS EC2 Instance you created above. This is a step by step process.
```
Note: Before proceeding be sure you delete the ingress, service, deployment, and namespace as instructed above.
```
### Delete the EKS Cluster using eksctl
Delete the EKS Cluster using eksctl
```
eksctl delete cluster --region us-east-1 --name=kubeflow
```
Wait till completed before proceeding.  

### Delete EC2 Instance
#### AWS EC2 Dashboard
Using the AWS Console goto the EC2 Dashboard and delete the ec2 instance we used as the eks_cloud_shell.
```
Terminate "eks_cloud_shell" Instance  
```

## References
Kubeflow
https://www.kubeflow.org/docs

Kubeflow on AWS
https://awslabs.github.io/kubeflow-manifests



