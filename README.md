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

root@master:~# kubectl get pods --all-namespaces     // To get info on all namespaces
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

#### Kubernetes Architecture: 

* Master node
* Worker node   (Where containers are hosted)

Kubelet

Performs health checks.
One who creates containers in the backend

ETCD

* Database for K8



--------------------------------------------------  DAY 3: --------------------------------------------------

### POD

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

As we can see a POD is created in  worker1 & another in worker2. Here scheduler decides to which node the POD must be created. 

Deleting a POD

```
root@master:~# kubectl delete pods nginx
pod "nginx" deleted
root@master:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
myapp   1/1     Running   0          2m45s
root@master:~#
```

To get more details about a POD, 

kubectl describe pod myapp


#### Creating a POD with YAML:

| Kind | Version/api |
| --- | --- |		
| Pod | v1 |
| service | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

Example yaml file: demo-app.yaml

To run:

kubectl create -f demo-app.yaml
kubectl get pods -o wide

#### Additional: --dry-run

* You can use the --dry-run=client flag to preview the object that would be sent to your cluster, without really submitting it. 
* Always use --dry-run when you want to generate any yaml file for a running pod or when creating one (if you are testing). 
* It displays any errors before actually executing the actual run cmd. 

```
root@master:~# kubectl run myapp --image=nginx:latest --dry-run=client
pod/myapp created (dry run)
root@master:~# kubectl get pods
No resources found in default namespace.
root@master:~#
```

Usecase 2: If you would like to create/generate an yaml file for above example, 

```
root@master:~# kubectl run myapp --image=nginx:latest --dry-run=client -o yaml > demo-pod.yaml
root@master:~# less demo-pod.yaml
```


--------------------------------------------------  DAY 4: --------------------------------------------------

### ReplicationController

* High availability
* Scaling can be done
* Loadbalancing setup
* Adding your pod to ReplicationController will make sure to create new/multiple replicas of pod automatically. 

```
apiVersion: v1
kind: ReplicationController
metadata:		  #Metadata & labels for RC
  name: nginx
  labels:
    app: myapp
    type: front-end
spec:                     #ReplicaController spec
  replicas: 3             #How many replicas to create
  template:		  #Template yaml for your POD creation
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

### ReplicaSet

* The main difference or feature about this is in the 'selector'. It supports lot of customization than the RC. 
* Selector > creates replicas based on labels. 
* A ReplicaSet ensures that a specified number of pod replicas are running at any given time.

```
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

* Here replicas will be created bases on label 'type:front-end'. If already there's any existing POD which has a label 'type:front-end' then the ReplicaSet will only create two replicas instead of 3. 

kubectl create -f your.yaml (replicaset yaml file)
kubectl get replicaset 

### Manual Scaling

To increase any replicas for any exisisting project/POD. Example if you suddenly decide that replicas for POD should be 6 instead of 3, then you can achive this by multiple methods. 

Method 1: By re-editing the yaml file which you used to create POD. Increase the replica field from 3 to 6. Then do, 

```
kubectl replace -f your.yaml
kubectl get pods -o wide
```

Method 2: Changing replica values without having to edit your yaml file. 
```
kubectl scale -replicas=6 -f your.yaml --record

root@master:~# kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
rs-demo.example.com   6         6         6       5m55s
```

Additionally you can add '--record' line to above cmd to record your changes to logs. 

Method 3: If we have to create a replicaset of 10 pods. Modifying a live or running POD.
```
kubectl edit replicaset.apps myapp-rs
```

To backup a configuration of a replicaset running in a cluster. 
```
kubectl get rs  -o yaml > nameof.yaml

or

kubectl get rs myapp-rs -o yaml > > nameof.yaml
```

--------------------------------------------------  DAY 5: --------------------------------------------------

### Deployment
* More features than replicaset
* Rollback support
* Rolling update possible
* It includes replicaset (The pods will be created inside the replicaset here)
* A Deployment runs multiple replicas of your application and automatically replaces any instances that fail or become unresponsive.
* A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features.

```
root@master:~# kubectl create -f deployment-example.yaml

root@master:~# kubectl get deployment.apps
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment-demo.example.com   3/3     3            3           73s

root@master:~# kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
deployment-demo.example.com-57686d7bf4-g6jpq   1/1     Running   0          46s
deployment-demo.example.com-57686d7bf4-qc4l6   1/1     Running   0          46s
deployment-demo.example.com-57686d7bf4-rtx7n   1/1     Running   0          46s
```

To see the ReplicaSet (rs) created by the Deployment, run kubectl get rs.
```
root@master:~# kubectl get rs
NAME                                     DESIRED   CURRENT   READY   AGE
deployment-demo.example.com-57686d7bf4   3         3         3       3m52s

root@master:~# kubectl get rc           // No ReplicaController created. Deployment only creates RS
No resources found in default namespace.

root@master:~# kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
deployment-demo.example.com-57686d7bf4-g6jpq   1/1     Running   0          3m59s
deployment-demo.example.com-57686d7bf4-qc4l6   1/1     Running   0          3m59s
deployment-demo.example.com-57686d7bf4-rtx7n   1/1     Running   0          3m59s
```

#### To see the labels automatically generated for each Pod,
```
root@master:~# kubectl get pods --show-labels
NAME                                           READY   STATUS    RESTARTS   AGE     LABELS
deployment-demo.example.com-57686d7bf4-g6jpq   1/1     Running   0          5m29s   app=myapp3,pod-template-hash=57686d7bf4,type=front-end
deployment-demo.example.com-57686d7bf4-qc4l6   1/1     Running   0          5m29s   app=myapp3,pod-template-hash=57686d7bf4,type=front-end
deployment-demo.example.com-57686d7bf4-rtx7n   1/1     Running   0          5m29s   app=myapp3,pod-template-hash=57686d7bf4,type=front-end
```

To check the labels for deployment/replicaset,
```
root@master:~# kubectl get deployment.apps --show-labels
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
deployment-demo.example.com   3/3     3            3           7m18s   app=myapp3-deployment-demo,type=front-end
```

### Namespaces

* Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.
* Helpful when different teams or projects share a Kubernetes cluster.
* Any number of namespaces are supported within a cluster, each logically separated from others but with the ability to communicate with each other.
* Project isolation can be done using Namespaces (Example namespace for adminstrators[web] & namespace for devs[mysql, db])
* Namespaces are a way to divide cluster resources between multiple users (via resource quota).
```
root@master:~# kubectl get namespaces
NAME              STATUS   AGE
default           Active   3d2h
kube-node-lease   Active   3d2h
kube-public       Active   3d2h
kube-system       Active   3d2h
```

To see pods created in specific namespaces, 
```
root@master:~# kubectl get pods --namespace default
NAME                                           READY   STATUS    RESTARTS   AGE
deployment-demo.example.com-57686d7bf4-g6jpq   1/1     Running   0          15m
deployment-demo.example.com-57686d7bf4-qc4l6   1/1     Running   0          15m
deployment-demo.example.com-57686d7bf4-rtx7n   1/1     Running   0          15m

root@master:~# kubectl get pods --namespace kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-4jj4z         1/1     Running   4          3d2h
coredns-558bd4d5db-qjx56         1/1     Running   4          3d2h
etcd-master                      1/1     Running   5          3d2h
kube-apiserver-master            1/1     Running   7          3d2h
kube-controller-manager-master   1/1     Running   27         3d2h
kube-proxy-ljjbp                 1/1     Running   4          3d2h
kube-proxy-rljj5                 1/1     Running   4          3d2h
kube-proxy-zb2t7                 1/1     Running   5          3d2h
kube-scheduler-master            1/1     Running   27         3d2h
weave-net-brxk9                  2/2     Running   11         3d2h
weave-net-bsgkx                  2/2     Running   9          3d2h
weave-net-gt2mn                  2/2     Running   9          3d2h
```

Creating a namespace
```
root@master:~# kubectl create namespace dev
namespace/dev created

root@master:~# kubectl run nginx --image=nginx --namespace=dev
pod/nginx created

root@master:~# kubectl get pods --namespace=dev
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          14s
```

More details:
https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

### Resource Quotas
* When several users or teams share a cluster with a fixed number of nodes, there is a concern that one team could use more than its fair share of resources.
* Resource quotas are a tool for administrators to address this concern.
* Different teams work in different namespaces. 
* The administrator creates one ResourceQuota for each namespace.
* The name of a ResourceQuota object must be a valid DNS subdomain name.

More details: https://kubernetes.io/docs/concepts/policy/resource-quotas/

Example: One namespace for 'development team' & another name for 'Production'. Suppose you only want to give 2GB mem for development team & 4Gb mem for Production team. This can be acheived using creating two different namespaces for dev & prod - then set resource quota.


```
root@master:~# kubectl describe ns default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.

root@master:~# kubectl describe ns myns
Name:         myns
Labels:       kubernetes.io/metadata.name=myns
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

Getting man page for ResourceQuota in Kubernetes

```
root@master:~# kubectl explain resourcequota
```

Now we can create a yaml file and set resource quota and limits for any given namespaces. 

```
root@master:~# kubectl create ns dev
namespace/dev created
root@master:~# kubectl create -f compute-quota.yaml
resourcequota/compute-quota created
root@master:~# kubectl describe ns dev
Name:         dev
Labels:       kubernetes.io/metadata.name=dev
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            compute-quota
  Resource         Used  Hard
  --------         ---   ---
  limits.cpu       0     6
  limits.memory    0     1Gi
  pods             0     5
  requests.cpu     0     3
  requests.memory  0     900m

No LimitRange resource.

OR

root@master:~# kubectl get resourcequotas -n dev
NAME            AGE     REQUEST                                                 LIMIT
compute-quota   3m28s   pods: 0/5, requests.cpu: 0/3, requests.memory: 0/900m   limits.cpu: 0/6, limits.memory: 0/1Gi
```

* limits.cpu	Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value.
* requests.cpu	Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value.


```
root@master:~# kubectl describe ns dev
Name:         dev
Labels:       kubernetes.io/metadata.name=dev
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            compute-quota
  Resource         Used  Hard
  --------         ---   ---
  limits.cpu       0     600m
  limits.memory    0     1Gi
  pods             0     5
  requests.cpu     0     500m
  requests.memory  0     900m

No LimitRange resource.
root@master:~#
root@master:~# kubectl get resourcequota -n dev
NAME            AGE   REQUEST                                                    LIMIT
compute-quota   23m   pods: 0/5, requests.cpu: 0/500m, requests.memory: 0/900m   limits.cpu: 0/600m, limits.memory: 0/1Gi
root@master:~#
root@master:~# kubectl create -f pod-limit-example.yaml
Error from server (Forbidden): error when creating "pod-limit-example.yaml": pods "myapp-pod-limit" is forbidden: exceeded quota: compute-quota, requested: requests.memory=150Mi, used: requests.memory=0, limited: requests.memory=900m
```

Delete all resourcequota

```
root@master:~# kubectl delete ResourceQuota --all
```

-------------------------------------------------- DAY 6: --------------------------------------------------

### Services

* A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects.
* It has a name, IP
* Each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service. Services allow your applications to receive traffic. Services can be exposed in different ways by specifying a type in the ServiceSpec:

##### Types:

Cluster IP
NodePort
Load balancer

### Cluster IP

* Accessing the content within the cluster

```
root@master:~# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d22h

Creating a POD & Accessing via port within cluster:

root@master:~# kubectl create -f pod-creation-example.yaml
pod/pod-creation.example.com created

root@master:~# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
pod-creation.example.com   1/1     Running   0          2m15s

root@master:~# kubectl expose pod pod-creation.example.com --port=8000 --target-port=80 --name myfirstservice
service/myfirstservice exposed

root@master:~# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP    7d23h
myfirstservice   ClusterIP   10.100.118.226   <none>        8000/TCP   7s

root@master:~# kubectl describe svc myfirstservice
Name:              myfirstservice
Namespace:         default
Labels:            app=myapp1
                   type=front-end
Annotations:       <none>
Selector:          app=myapp1,type=front-end
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.100.118.226
IPs:               10.100.118.226
Port:              <unset>  8000/TCP
TargetPort:        80/TCP
Endpoints:         10.38.0.1:80
Session Affinity:  None
Events:            <none>

root@master:~# curl http://10.100.118.226:8000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```

### NodePort

* A NodePort is an open port on every node of your cluster.
* Kubernetes transparently routes incoming traffic on the NodePort to your service, even if your application is running on a different node.
* NodePort is assigned from a pool of cluster-configured NodePort ranges (typically 30000–32767).

![Screenshot 2021-07-01 at 6 09 15 PM](https://user-images.githubusercontent.com/29716063/124125752-99316480-da97-11eb-8933-3e77285ce341.png)

To link the above service to any POD, we can do it using selector & label.

![Screenshot 2021-07-01 at 6 11 20 PM](https://user-images.githubusercontent.com/29716063/124125964-cda52080-da97-11eb-86a1-e1b6a41130d3.png)

Now create the service,

![Screenshot 2021-07-01 at 6 12 44 PM](https://user-images.githubusercontent.com/29716063/124126188-0d6c0800-da98-11eb-8adb-5f23cdefdf16.png)


Exposing a NodePort via adhoc instead of using yaml. 

```
root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
pod-creation.example.com   1/1     Running   0          15m   10.38.0.1   worker2   <none>           <none>

root@master:~# kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE     SELECTOR
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP    7d23h   <none>
myfirstservice   ClusterIP   10.100.118.226   <none>        8000/TCP   11m     app=myapp1,type=front-end

root@master:~# kubectl expose pod pod-creation.example.com --type=NodePort --port=8000 --target-port=80 --name myfirstservice1
service/myfirstservice1 exposed

root@master:~# kubectl get svc -o wide
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE     SELECTOR
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          7d23h   <none>
myfirstservice    ClusterIP   10.100.118.226   <none>        8000/TCP         13m     app=myapp1,type=front-end
myfirstservice1   NodePort    10.109.133.56    <none>        8000:32689/TCP   8s      app=myapp1,type=front-end
```

Now we can access our app using the NodeIP:32689 port mention in above snippet. 

![Screenshot 2021-07-01 at 7 15 08 PM](https://user-images.githubusercontent.com/29716063/124135053-157c7580-daa1-11eb-96a1-d62a1dce5e0f.png)

### LoadBalancer

To be updated...!!!!

### How services work?

![Screenshot 2021-07-01 at 6 01 44 PM](https://user-images.githubusercontent.com/29716063/124124841-8b2f1400-da96-11eb-9adc-dcd73fd4248e.png)

Create a POD first and should have a label:

```
root@master:~# kubectl create -f pod-creation-example.yaml
pod/pod-creation.example.com created

root@master:~# kubectl get pods pod-creation.example.com --show-labels
NAME                       READY   STATUS    RESTARTS   AGE   LABELS
pod-creation.example.com   1/1     Running   0          85s   app=myapp1,type=front-end
```

Incase you forgot to set label for POD, then you can add label later using,
```
root@master:~# kubectl label pod pod-creation.example.com gender=male
pod/pod-creation.example.com labeled

root@master:~# kubectl get pods pod-creation.example.com --show-labels
NAME                       READY   STATUS    RESTARTS   AGE   LABELS
pod-creation.example.com   1/1     Running   0          13m   app=myapp1,gender=male,type=front-end
```

Now create a service and link it to POD using any labels,
```
root@master:~# kubectl create -f myfirstservice.yaml
service/myfirstservice created

root@master:~# kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE     SELECTOR
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          7d23h   <none>
myfirstservice   NodePort    10.97.120.226   <none>        8000:32000/TCP   56s     type=front-end

root@master:~# curl http://3.23.114.147:32000/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
```

### Scheduling

#### Manual Scheduling

* To create a pod in any given Node. (To force creation of a pod to your favorite)

```
root@master:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   8d    v1.21.2
worker1   Ready    <none>                 8d    v1.21.2
worker2   Ready    <none>                 8d    v1.21.2

root@master:~# kubectl get pods
No resources found in default namespace.

root@master:~# kubectl create -f manual-shedule-deployment.yaml
deployment.apps/manual-shedule-deployment created

root@master:~# kubectl get deployments.apps
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
manual-shedule-deployment   3/3     3            3           26s

root@master:~# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
manual-shedule-deployment-7ffdc579cc-kssk6   1/1     Running   0          28s   10.38.0.2   worker2   <none>           <none>
manual-shedule-deployment-7ffdc579cc-pccfw   1/1     Running   0          28s   10.38.0.1   worker2   <none>           <none>
manual-shedule-deployment-7ffdc579cc-w7mmv   1/1     Running   0          28s   10.38.0.3   worker2   <none>           <none>
```
As you can see all pods are created in worker2 as given in yaml file (manual-shedule-deployment.yaml). 

-------------------------------------------------- DAY 7: --------------------------------------------------



