# kubernetes-zynux

### Lab Setup:

1) Launch '3' t2 instances on AWS with security groups 'default (VPC)' & 'custom group (for external port accesses)'. 

2) Name one servers as 'master', second server as 'worker1', third server as 'worker2'

Once instances are setup for forming a cluster, proceed with docker installation & Kubeadm setup. 

Step 1: Install Docker CE

Installation for Ubuntu, as per:
https://docs.docker.com/engine/install/ubuntu/

curl -o- https://raw.githubusercontent.com/heyitsak/kubernetes-zynux/main/docker-ce-install.sh | bash 

Step 2: 

Following instructions outlined below doc to setup & install Kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
