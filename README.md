# k8s
k8s_documentations

Steps by Steps k8s Installation on AWS using kops:

1) Launch Linux EC2 instance in AWS (for k8s Client)

2) Creating IAM Role for ec2 to create aws resource and attache Administrator policy as of now
Note: Kops need permissions to access
    AmazonEC2FullAccess
    AmazonRoute53FullAccess
    AmazonS3FullAccess
    IAMFullAccess
    AmazonVPCFullAccess


3) Install kops on ec2:
# wget https://github.com/kubernetes/kops/releases/download/v1.18.0-alpha.3/kops-linux-amd64

# chmod +x kops-linux-amd64

# mv kops-linux-amd64 /usr/local/bin/kops

4) Install kubectl:
# curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

# chmod +x kubectl

# mv kubectl /usr/local/bin/kubectl

5) Create S3 bucket in AWS:
-> S3 bucket is used by kubernetes to persist cluster state, lets create s3 bucket using aws cli Note: Make sure you choose bucket name that is uniqe accross all aws accounts

# aws s3 mb s3://javahome.in.k8s --region ap-south-1   [we can create bucket using Console as well]

6) Create private hosted zone in AWS Route53 for k8s cluster
- Create hosted zone with any name what you want like skumark8s.in
- Choose type as privated hosted zone for VPC
- Select default vpc in the region you are setting up your cluster

7) Configure environment variables:
# vim .bashrc
export KOPS_CLUSTER_NAME=skumark8s.in
export KOPS_STATE_STORE=s3://skumark8s.in

# source .bashrc

8) Create ssh key pair
This keypair is used for ssh into kubernetes cluster

9) Create a Kubernetes cluster definition:
# kops create cluster --state=${KOPS_STATE_STORE} --node-count=2 --master-size=t2.micro \
--node-size=t2.micro --zones=us-east-1a,us-east-1b --name=${KOPS_CLUSTER_NAME} --dns private \
--master-count 1 

* list clusters with: kops get cluster
* edit this cluster with: kops edit cluster skumark8s.in
* edit your node instance group: kops edit ig --name=skumark8s.in nodes
* edit your master instance group: kops edit ig --name=skumark8s.in master-us-east-1a

10) Create kubernetes cluster
# kops update cluster --yes


* validate cluster: kops validate cluster
* list nodes: kubectl get nodes --show-labels
* ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.skumark8s.in
* the admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
* read about installing addons at: https://kops.sigs.k8s.io/operations/addons.


Note: Above command may take some time to create the required infrastructure resources on AWS. Execute the validate command to check its status and wait until the cluster becomes ready

# kops validate cluster

11) To connect to the master
# ssh admin@api.skumark8s.in


Optional:

* if you want to destroy the kubernetes cluster
# kops delete cluster  --yes
