# kubernetes-zynux

###### DAY 1: 

#### Creating a lab Setup for kubernetes cluster:

1) Launch '3' t2 instances on AWS with security groups 'default (VPC)' & 'custom group (for external port accesses)'. The default VPC group will make sure that all 3 instances can comunicate to each other. 

2) Name one servers as 'master', second server as 'worker1', third server as 'worker2'

Once instances are setup for forming a cluster, proceed with docker CE installation & setup Kubeadm on all 3 aws instances. 

#### Install Docker CE

Installation for Ubuntu, as per:
https://docs.docker.com/engine/install/ubuntu/

curl -o- https://raw.githubusercontent.com/heyitsak/kubernetes-zynux/main/docker-ce-install.sh | bash 

#### Installing kubeadm, kubelet and kubectl

Following instructions outlined below doc to setup & install Kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

curl -o- https://raw.githubusercontent.com/heyitsak/kubernetes-zynux/main/Kubeadm-install.sh | bash

###### DAY 2: 

Once the step1,2 is completed, we can start by running 'kubeadm init' to start creating a cluster. 

kubeadm init -h

* kubeadm init phase enables you to invoke atomic steps of the bootstrap process. Running 'kubeadm init' is the first step to initialize and to create a cluster. 
* It will auto install required containersto make a cluster. 

kubectl 

* To administer cluster

kubectl get 

kubeadm 

* Its like a user


