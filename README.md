AWS Elastic Kubernetes Service (EKS) Kubeflow QuickStart  
=======================================================
Abstract:
```
With the recent advent of integrated development environments (IDE) such as AWS SageMaker Studio
for Machine Learning (ML), it's now possible to provide a single integrated view where you can
access purpose-built tools to perform ML development tasks from preparing data to building,
training, and deploying your ML models.

Kubeflow is an open source ML IDE you can also access purpose-built tools to perform ML development
tasks from preparing data to building, training, and deploying your ML models, but it runs on
Kubernetes.  But because it runs on Kubernetes its portable and scalable across multiple cloud and
on premise infrastructure.

Please see: https://www.youtube.com/@kal_technology/videos

```
This solution shows how to create an AWS EKS Cluster with Kubeflow support.  
```
Note: This how-to assumes you are creating an eks cluster in us-east-1, you have access to your
AWS Account, and you can login to an EC2 Instance remotely.
```
Steps:  
* [Create kubeflow_cloud_shell](#Create-kubeflow_cloud_shell)
* [Connect to kubeflow_cloud_shell and Install Basic Tools](#Connect-to-kubeflow_cloud_shell-and-Install-Basic-Tools)
* [Set Kubeflow Version and Check Out Kubeflow Repo](#Set-Kubeflow-Version-and-Check-Out-Kubeflow-Repo)
* [Configure AWS CLI Security Credentials](#Configure-AWS-CLI-Security-Credentials)
* [Create EKS Cluster](#Create-EKS-Cluster)
* [Deploy Kubeflow](#Deploy-Kubeflow)
* [Connect to Kubeflow Dashboard](#Connect-to-Kubeflow-Dashboard)
* [Delete EKS Cluster](#Delete-EKS-Cluster)
* [Delete kubeflow_cloud_shell](#Delete-kubeflow_cloud_shell)
* [Troubleshooting](#Troubleshooting)
* [References](#References)

### Create kubeflow_cloud_shell
We'll using an EC2 instance to install awscli , eksctl, and other support tools needed to create
the EKS Cluster and Install Kubeflow.

#### Login to the AWS Managment Console go to the AWS EC2 Dashboard  
Click on "Launch Instance"  

Name and tags
Name:
```
kubeflow_cloud_shell
```
Application and OS Images (Amazon Machine Image)
```
NOTE: Please use the specific AMI.  I found that using the latest Ubuntu AMI broke due to
      Perl3.8 being deployed differently on later versions of Ubuntu.
```       
Search:
```
ubuntu-bionic-18.04-amd64-server-20220901
```
Choose AMI from Community AMI Tab
```
Canonical, Ubuntu, 18.04 LTS, amd64 bionic image build on 2022-09-01
```  
Click on "Select"

Instance Type
```
t2.micro
```
Key pair (login)
```
Your AWS Account Keypair
```
Network settings 
You could use the default security group if you've already added ssh as an inbound rule
```
Select "Create Security Group"
Select "Allow SSH traffic from"
Select "Anywhere"
```
Review and Launch  

### Connect to kubeflow_cloud_shell and Install Basic Tools

NOTE: AWS EC2 Key Pair File should be in the directory you run ssh from  
NOTE: You'll need AWS EC2 Public IPv4 DNS for your kubeflow_cloud_shell

```
ssh -i <AWS EC2 Key Pair File> ubuntu@<Public IPv4 DNS> -o ExitOnForwardFailure=yes

```

Install Basic Tools
```
sudo apt update
sudo apt install git curl unzip tar make sudo vim wget -y

```

### Set Kubeflow Version and Check Out Kubeflow Repo
This will checkout the Kubeflow project
```
export KUBEFLOW_RELEASE_VERSION=v1.6.1
export AWS_RELEASE_VERSION=main

git clone https://github.com/awslabs/kubeflow-manifests.git && cd kubeflow-manifests
git checkout ${AWS_RELEASE_VERSION}
git clone --branch ${KUBEFLOW_RELEASE_VERSION} https://github.com/kubeflow/manifests.git upstream

```
Install Kubeflow Specific Tools
```
make install-tools

```

Set the python alias and insure you have Python Version 3.8 installed.
```
alias python=python3.8
python
exit()

```

### Configure AWS CLI Security Credentials
Use the AWS CLI to set Access Key, Secret Key, and Region Name
```
aws configure --profile=kubeflow

```
AWS Access Key ID []: "Your Access Key ID"  
AWS Secret Access Key []: "Your Secret Access Key"   
Default region name []: us-east-1  
Default output format []: json

```
export AWS_PROFILE=kubeflow

```

Test AWS CLI to insure it has access to AWS Resources
```
aws s3 ls

```

### Create EKS Cluster 
Using eksctl to create the eks cluster (takes about 20 minutes)
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

Check Cluster status to insure the cluster status is "Active"
```
aws eks describe-cluster --region us-east-1 --name kubeflow --query "cluster.status"
```
Check Cluster Nodes Using kubectl test status of cluster nodes
```
kubectl get nodes
```
#### Deploy Kubeflow
Be sure you're in "kubeflow-manifests" directory before deploying kubeflow! (takes about 10 minutes)
```
cd $HOME/kubeflow-manifests
make deploy-kubeflow INSTALLATION_OPTION=helm DEPLOYMENT_OPTION=vanilla
```

Check kubeflow status
```
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n auth
kubectl get pods -n knative-eventing
kubectl get pods -n knative-serving
kubectl get pods -n kubeflow
kubectl get pods -n kubeflow-user-example-com

```

### Connect to Kubeflow Dashboard
Run the following command directly from your kubeflow_cloud_shell
```
make port-forward
```
Use a second ssh from your local machine to open a tunnel to your kubeflow_cloud_shell.  This second
ssh will allow you to view the Kubeflow Dashboard directly on your local machine.

NOTE: AWS EC2 Key Pair File should be in the directory you run ssh  
NOTE: Public IPv4 DNS for your kubeflow_cloud_shell
```
ssh -i <AWS EC2 Key Pair File> -L 8080:localhost:8080 -N ubuntu@<Public IPv4 DNS> -o ExitOnForwardFailure=yes
```
Connect to Kubeflow Dashboard using your Local Browser
Enter the following URL.
```
http://localhost:8080

```
Login to the Kubeflow Dashboard with the following credentials.
Email Address
Password
```
user@example.com

```
```
12341234

```
### Delete EKS Cluster
```
eksctl delete cluster --region us-east-1 --name=kubeflow
```
Wait till cluster is deleted before proceeding.  

### Delete kubeflow_cloud_shell
Using the AWS Console goto the EC2 Dashboard and delete the ec2 instance we used as the
eks_cloud_shell.
```
Terminate "kubeflow_cloud_shell" Instance  
```
### Troubleshooting
* "make deploy-kubeflow INSTALLATION_OPTION=helm DEPLOYMENT_OPTION=vanilla" Fails!  
Recommend removing and creating a new eks cluster, then deploy Kubeflow again.

### References
Kubeflow
https://www.kubeflow.org/docs  
Kubeflow on AWS
https://awslabs.github.io/kubeflow-manifests



