# kubernetes-zynux

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


--------------------------------------------------  DAY 2: --------------------------------------------------


Now we can start by running 'kubeadm init' to start creating a cluster. 

Note: If you are trying to create cluster using the 'kubeadm init' in AWS t2 instance, you might see some warning stating that the cluster requires a RAM of 2GB for proper functioning. Sin our case since its for studying purpose you can ignore those warining and proceed with creation of cluster by running 

kubeadm init --ignore-preflight-errors=all

(Run this on the master node and you should see somthing like this as output), 

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.24.182:6443 --token 7pang8.6fqhpz0hqw1fhpgn \
	--discovery-token-ca-cert-hash sha256:a308cd9bfedda0b21ced6197dede017f138c5512044f9aa63d74583fa8a987d4
```

Adding nodes to your cluster can be done using above kubeadm join token command in the worker nodes.

Once added run, 

```
root@master:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   22h   v1.21.2
worker1   NotReady    <none>                 22h   v1.21.2
worker2   NotReady    <none>                 22h   v1.21.2
```

As you can see we are able to see the worker nodes from master however the status of worker node shows 'NotReady'. This is because we need to setup CNI (Container Network Interface) in the nodes. This is like LAN hardware connection (Example - cilcum, dlink, weavenet) 

Why is this important? 

Its a networking toolkit. It creates a virtual network that connects Docker containers across multiple hosts and enables their automatic discovery.

Here we will setup weavenets in our master and worker nodes. To setup this you can follow instructions outlined in below documentation. 
https://www.weave.works/docs/net/latest/kubernetes/kube-addon/

Once we have installed the weave nets on our cluster, re-run the command to see if the node status is changed.

```
root@master:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   22h   v1.21.2
worker1   Ready    <none>                 22h   v1.21.2
worker2   Ready    <none>                 22h   v1.21.2
```
Now status shows 'Ready'

Notes:

kubeadm init -h

* kubeadm init phase enables you to invoke atomic steps of the bootstrap process. Running 'kubeadm init' is the first step to initialize and to create a cluster. 
* It will auto install all containers that are required to create a cluster. 

kubectl 

* To administer cluster
* allows you to run commands against Kubernetes clusters
* To deploy applications, inspect and manage cluster resources, and view logs.

kubeadm 

* Its like a user
* Kubeadm is a tool built to provide kubeadm init and kubeadm join as best-practice "fast paths" for creating Kubernetes clusters
* it runs on each node, and basically creates and then talks to the Kubernetes API.

Namespaces 

Namespaces are a way to organize clusters into virtual sub-clusters — they can be helpful when different teams or projects share a Kubernetes cluster. Any number of namespaces are supported within a cluster, each logically separated from others but with the ability to communicate with each other.

```
root@master:~# kubectl get namespaces
NAME              STATUS   AGE
default           Active   23h
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h      // All services required for K8 to work 

root@master:~# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-4jj4z         1/1     Running   1          23h
coredns-558bd4d5db-qjx56         1/1     Running   1          23h
etcd-master                      1/1     Running   2          23h
kube-apiserver-master            1/1     Running   2          23h
kube-controller-manager-master   1/1     Running   8          23h
kube-proxy-ljjbp                 1/1     Running   1          22h
kube-proxy-rljj5                 1/1     Running   1          22h
kube-proxy-zb2t7                 1/1     Running   2          23h
kube-scheduler-master            1/1     Running   8          23h
weave-net-brxk9                  2/2     Running   5          23h
weave-net-bsgkx                  2/2     Running   3          22h
weave-net-gt2mn                  2/2     Running   3          22h
root@master:~#
```

You can also add '-o wide' to your command to show extended details such as node details and IP's etc. 

```
root@master:~# kubectl get pods -n kube-system -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP              NODE      NOMINATED NODE   READINESS GATES
coredns-558bd4d5db-4jj4z         1/1     Running   1          23h   10.32.0.3       master    <none>           <none>
coredns-558bd4d5db-qjx56         1/1     Running   1          23h   10.32.0.2       master    <none>           <none>
etcd-master                      1/1     Running   2          23h   172.31.24.182   master    <none>           <none>
kube-apiserver-master            1/1     Running   2          23h   172.31.24.182   master    <none>           <none>
kube-controller-manager-master   1/1     Running   8          23h   172.31.24.182   master    <none>           <none>
kube-proxy-ljjbp                 1/1     Running   1          22h   172.31.17.187   worker1   <none>           <none>
kube-proxy-rljj5                 1/1     Running   1          22h   172.31.43.221   worker2   <none>           <none>
kube-proxy-zb2t7                 1/1     Running   2          23h   172.31.24.182   master    <none>           <none>
kube-scheduler-master            1/1     Running   8          23h   172.31.24.182   master    <none>           <none>
weave-net-brxk9                  2/2     Running   5          23h   172.31.24.182   master    <none>           <none>
weave-net-bsgkx                  2/2     Running   3          22h   172.31.43.221   worker2   <none>           <none>
weave-net-gt2mn                  2/2     Running   3          22h   172.31.17.187   worker1   <none>           <none>
```

Notes: 

N/w policy are stored in --> Kube-proxy

##### Kubernetes Architecture: 

* Master node
* Worker node   // Where containers are hosted

Kubelet

Performs health checks.
One who creates containers in the backend

ETCD

* Database for K8



--------------------------------------------------  DAY 3: --------------------------------------------------

#### POD

* Single/multiple applications on POD
* It contains container (one or more)
* POD works under Nodes (Ex: Master, worker node)
* IP address is assigned to PODS and not to containers

Creating a POD:

```
root@master:~# kubectl run nginx --image nginx
pod/nginx created
root@master:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s
```

Here 'READY' shows '1/1' --> It means only one container is running in the pod

Creating a POD for my new application called 'myapp'

```
root@master:~# kubectl run myapp --image nginx:latest
pod/myapp created

root@master:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          9s
nginx   1/1     Running   0          2m2s

root@master:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
myapp   1/1     Running   0          18s     10.38.0.1   worker2   <none>           <none>
nginx   1/1     Running   0          2m11s   10.36.0.0   worker1   <none>           <none>
```

Deleting a POD

```
root@master:~# kubectl delete pods nginx
pod "nginx" deleted
root@master:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          2m45s
root@master:~#
```

--------------------------------------------------  DAY 4: --------------------------------------------------


