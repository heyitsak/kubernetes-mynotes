# kubernetes-mynotes

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
### Advanced Scheduling
#### Taint and toleration

Taints:  
* they allow a node to repel a set of pods.
* Think of this as some filter set on node

Toleration:
* Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Examples:
* No pods get created to master
* Because 'taint' is enabled in master node
* We can remove the 'taint' via configuration then the pods can be created in master or specified node
* We can set 'taint' for any node

```
root@master:~# kubectl describe node master | grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule
root@master:~#
```
You add a taint to a node using kubectl taint. For example,

```
kubectl taint nodes node1 key1=value1:NoSchedule
```
* places a taint on node `node1`. The taint has key `key1`, value `value1`, and taint effect `NoSchedule`. 
* This means that no pod will be able to schedule onto `node1` unless it has a matching toleration.

To remove the taint added by the command above, you can run:

```
kubectl taint nodes node1 key1=value1:NoSchedule-
```

#### Setting toleration for pods

Specify toleration for a pod in the `PodSpec`. Both of the following tolerations "match" the taint created by the `kubectl taint` line above, and thus a pod with either toleration would be able to schedule onto `node1`:

More info https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

##### Example: 

```
root@master:~# kubectl describe node worker1 | grep -i taints
Taints:             <none>
root@master:~# kubectl describe node worker2 | grep -i taints
Taints:             <none>
```
Setting filter/taint for Worker1
```
root@master:~# kubectl taint node worker1 mysize=large:NoSchedule
node/worker1 tainted

root@master:~# kubectl describe node worker1 | grep -i taints
Taints:             mysize=large:NoSchedule

root@master:~# kubectl create -f pod-creation-example.yaml
pod/pod-creation.example.com created
```
The Pod got created in `Worker2` as the "pod-creation-example.yaml" didnt have any toleration set to work in `Worker1`.
```
root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
pod-creation.example.com   1/1     Running   0          5m    10.38.0.1   worker2   <none>           <none>
```
Now creating a Pod which has toleration set and which can work in both Worker1 or Worker2. 

```
root@master:~# kubectl create -f pod-creation-toleration.yaml
pod/tolerationpod created

root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
pod-creation.example.com   1/1     Running   0          15m     10.38.0.1   worker2   <none>           <none>
tolerationpod              1/1     Running   0          5m42s   10.36.0.0   worker1   <none>           <none>
```
As you can see, here `tolerationpod ` got created in Worker1 node. 

```
root@master:~# kubectl taint nodes worker1 mysize=large:NoSchedule-
node/worker1 untainted
root@master:~# kubectl describe node worker1 | grep -i taints
Taints:             <none>
```

-------------------------------------------------- DAY 8: --------------------------------------------------

#### NodeSelector Scheduling

* This is used when we want to only schedule particular pods to given nodes
* Here scheduling is done using `labels`
* When scheduling no conditions can be given in yaml [limitation of NodeSelector]
* **For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well). The most common usage is one key-value pair.**

Steps: 

```
root@master:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   12d   v1.21.2
worker1   Ready    <none>                 12d   v1.21.2
worker2   Ready    <none>                 12d   v1.21.2

root@master:~# kubectl get nodes --show-labels
NAME      STATUS   ROLES                  AGE   VERSION   LABELS
master    Ready    control-plane,master   12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
worker1   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux
worker2   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker2,kubernetes.io/os=linux
```

Assigning label for `worker1` node,
```
root@master:~# kubectl label nodes worker1 size=large
node/worker1 labeled

root@master:~# kubectl get nodes --show-labels | grep worker1
worker1   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux,size=large
```

Now create pod which should match the label for `worker1` node `size=large`,

Yaml file for pod,
```
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
  nodeSelector:
    size: "large"
```
Creating a pod,
```
root@master:~# kubectl create -f pod-creation-nodeselector.yaml
pod/pod-creation.example.com created
```
Now you can see that the pod is created to `worker1` node as we wanted,
```
root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
pod-creation.example.com   1/1     Running   0          19s   10.36.0.0   worker1   <none>           <none>
```
Now, to remove the label set for node,
```
root@master:~# kubectl get nodes --show-labels | grep worker1
worker1   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux,size=large

root@master:~# kubectl label nodes worker1 size-
node/worker1 labeled

root@master:~# kubectl get nodes --show-labels | grep worker1
worker1   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux
```
#### Node Affinity Scheduling

* This overcome the limitation of NodeSelector
* Conditions can be set 
* Suppose the label for pod doesn't match any of the labels on nodes, then the pod creation will goto pending and will not created. So to avoid such situation we need to write conditions. 
* Node affinity is a set of rules used by the scheduler to determine where a pod can be placed. 
* The rules are defined using custom labels on nodes and label selectors specified in pods.
* For example, you could configure a pod to only run on a node with a specific CPU or in a specific availability zone.

Two types of scheduling can be done here, 

#### Soft & Hard

##### Soft: 
This will try to create a pod matching a label in node. So incase if any node doesn't match the given label then go and schedule in any other nodes.

`preferredDuringSchedulingIgnoredDuringExecution`

##### Hard: 
Only create the pod matching label of particular node. 

`RequiredDuringSchedulingIgnoreduringExecution`

#### Example for `soft scheduling`, 

Set label for two nodes `worker1` & `worker2`, 
```
root@master:~# kubectl label node worker1 size=large
node/worker1 labeled
root@master:~# kubectl label node worker2 size=small
node/worker2 labeled

root@master:~# kubectl get nodes --show-labels | egrep "worker1|worker2"
worker1   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux,size=large

worker2   Ready    <none>                 12d   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker2,kubernetes.io/os=linux,size=small
```
Now define `Node Affinity` for pod in yaml file,
```
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
  affinity:
   nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - preference:
       matchExpressions:
       - key: size
         operator: In
         values:
         - small
      weight: 1
```
Create & run yaml file,
```
root@master:~# kubectl create -f pod-creation-nodeaffinity-soft.yaml
pod/pod-creation.example.com created

root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
pod-creation.example.com   1/1     Running   0          67s   10.38.0.1   worker2   <none>           <none>
```
Here you can see that the pod is created on Worker2. 

Now to understand this, try creating a pod which will not match any label (small or large) for nodes and see what happens,

Example pod yaml, 
```
  affinity:
   nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - preference:
       matchExpressions:
       - key: size
         operator: In
         values:
         - extralarge
```
Here, I have set label as `extralarge` for pod, which obivously doesnt match any of the worker nodes labels.
```
root@master:~# kubectl create -f pod-creation-nodeaffinity-soft-testing.yaml
pod/pod-test.example.com created

root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
pod-creation.example.com   1/1     Running   0          7m25s   10.38.0.1   worker2   <none>           <none>
pod-test.example.com       1/1     Running   0          84s     10.36.0.0   worker1   <none>           <none>
```
Here you can see that the second pod `pod-test.example.com ` got created in `worker1` anyway even if the label doesnt match any of the worker nodes. 

#### Example for `hard scheduling`

`RequiredDuringSchedulingIgnoreduringExecution`

Here I tried to create a pod with nodeaffinity matching label `extralarge`, [Note: we know that there are no nodes with label set as `extralarge`]
```
  affinity:
   nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
     nodeSelectorTerms:
     - matchExpressions:
       - key: size
         operator: In
         values:
         - extralarge
```
Lets see what happens here,
```
root@master:~# kubectl create -f pod-creation-nodeaffinity-hard.yaml
pod/pod-creation.example.com created

root@master:~# kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP       NODE     NOMINATED NODE   READINESS GATES
pod-creation.example.com   0/1     Pending   0          2m21s   <none>   <none>   <none>           <none>
```

The pod creation is in pending state here, let see why?
```
root@master:~# kubectl describe pod pod-creation.example.com

Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  2m35s  default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 node(s) didn't match Pod's node affinity/selector.
root@master:~#
```

### Resource Limits Scheduling

* Think of nodes like servers, each node has thier own resources for `CPU, MEM, DISK`. 
* And every pod consumes a set of resources on the nodes
* If the nodes doesn't have enough resources available - then the kube-scheduler will not allocate new pods to it [You will see the pod status will be in pending state]. 

Resource requirement for each pods:

* By default kubernetes assumes that a pod and container within a pod requires `CPU (0.5), MEM (256Mi)`, this is know as the `resource requests` for a container. The minimum amount of CPU or MEM requested by a container. 

* If your applications require more than these limits, you can modify the pod definition file. 

Resource requests for pod can specified as,
```
resources:
 requests:
  memory: "1Gi"
  cpu: "1"
```
By any chance if pod try to utlize more resources, the kubernetes by default has set a resource limit of `1vCPU` & `512Mi` for pod usage. 

Resource limits for pod can specified as,
```
resources:
 requests:
  memory: "1Gi"
  cpu: "1"
 limits:
  memory: "2Gi"
  cpu: "2"
```
So here when the pod is created using the above definition. The kubernetes sets new limits for the container. **Remember the limits and requests are set for each containers within the pod.**

Exceed Limits: What if any pod try over use CPU or MEM? 

Kubernets won't allow the pod to overuse the given CPU limit. However the case for memory is different, incase the pod constantly overuses memory on node then the pod will get auto terminated.


### DaemonSets Scheduling

* Daemon sets are like replica sets, like they run multiple instance of pod in nodes. 
* The main difference here is, the daemon set only creats one copy of pod in each node. 
* And whenever a new node is added to the cluster a replica of pod is placed into that node.
* **It ensures that always one copy of pod is present in all nodes in the cluster**

Use cases:

* Say if you want to deploy a `monitoring solution` or `log viewer/collector` on your cluster so you can monitor your each of your nodes in cluster better. Here you dont need to worry about adding or removing monitoring agent from cluster, as daemon set will take care of this for you. 

* Kube-proxy component can be example of daemon set. As this can also be deployed using daemon set. 
* N/w area, example weave network on each nodes

Examples with DaemonSets,

How many DaemonSets are created in the cluster in all namespaces?
```
root@controlplane:~# kubectl get daemonsets
No resources found in default namespace.
root@controlplane:~# kubectl get daemonsets --all-namespaces
NAMESPACE     NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-flannel-ds   1         1         1       1            1           <none>                   7m2s
kube-system   kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   7m7s
```
Find the image used by the POD deployed by the kube-flannel-ds DaemonSet?
```
root@controlplane:~# kubectl describe daemonsets -n kube-system kube-flannel-ds | grep -i image
    Image:      quay.io/coreos/flannel:v0.13.1-rc1
    Image:      quay.io/coreos/flannel:v0.13.1-rc1
```
Now deploy a DaemonSet for FluentD Logging,

An easy way to create a DaemonSet is to first generate a YAML file for a Deployment with the command,
```
root@controlplane:~# kubectl create deployment elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml
```
Next, remove the replicas and strategy fields from the YAML file using a text editor. Also, change the kind from Deployment to DaemonSet. And finally, create the Daemonset by running kubectl create -f fluentd.yaml
```
root@controlplane:~# kubectl create -f fluentd.yaml
daemonset.apps/elasticsearch created

root@controlplane:~# kubectl get daemonset -n kube-system
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
elasticsearch     1         1         0       1            0           <none>                   41s
kube-flannel-ds   1         1         1       1            1           <none>                   21m
kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   21m
```
-------------------------------------------------- DAY 9: --------------------------------------------------

### Static Node

* Static pods are usually used by software bootstrapping kubernetes itself. 
* For example, kubeadm uses static pods to bringup kubernetes control plane components like api-server, controller-manager as static pods.
* A kubelet can manage a node idependently 
* Static Pods are managed directly by the kubelet daemon on a specific node, without the API server 

Consider if there is no master node or controlpane, then how will you create a pod on a worker node with only kubelet present, 

* You can configure a kubelet to read the pod definition files from a directory `/etc/kubernetes/manifests`.
* Kubelet constantly monitors this directory for pod definition and makes changes to pods. 
* If we remove a file from this directory, the pod is removed automatically.
* We can only create pod using pod definition in this dir. 
* We cant create any replicaset or deployments or any other from this dir. 
* Kubelet works at pod level and can only understand pods.
* **So basically we can say that the pods created from this directoryby kublet without the intervention of api server are called `static pods`.**

This could be any directory `/etc/kubernetes/manifests`, so how do we configure one? 

![Screenshot 2021-07-07 at 7 25 38 PM](https://user-images.githubusercontent.com/29716063/124771970-36d5d980-df59-11eb-91c1-efa933e9a93d.png)

Another method using `--config path`

![Screenshot 2021-07-07 at 7 27 36 PM](https://user-images.githubusercontent.com/29716063/124772255-80262900-df59-11eb-928c-3466a3a924de.png)

Example, 

How many static pods exist in this cluster in all namespaces?

Run the command `kubectl get pods --all-namespaces` and look for those with `-controlplane` appended in the name

```
root@controlplane:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-bn2l8                1/1     Running   0          6m12s
kube-system   coredns-74ff55c5b-t8n74                1/1     Running   0          6m9s
kube-system   etcd-controlplane                      1/1     Running   0          6m15s
kube-system   kube-apiserver-controlplane            1/1     Running   0          6m15s
kube-system   kube-controller-manager-controlplane   1/1     Running   0          6m15s
kube-system   kube-flannel-ds-s799x                  1/1     Running   0          6m13s
kube-system   kube-flannel-ds-stskc                  1/1     Running   0          5m19s
kube-system   kube-proxy-fl68w                       1/1     Running   0          5m20s
kube-system   kube-proxy-sscnq                       1/1     Running   0          6m13s
kube-system   kube-scheduler-controlplane            1/1     Running   0          6m15s
```
Or 
```
root@controlplane:~# cd /etc/kubernetes/manifests/
root@controlplane:/etc/kubernetes/manifests# ll
total 28
drwxr-xr-x 1 root root 4096 Jul  7 13:54 ./
drwxr-xr-x 1 root root 4096 Jul  7 13:54 ../
-rw------- 1 root root 2177 Jul  7 13:54 etcd.yaml
-rw------- 1 root root 3802 Jul  7 13:54 kube-apiserver.yaml
-rw------- 1 root root 3314 Jul  7 13:54 kube-controller-manager.yaml
-rw------- 1 root root 1384 Jul  7 13:54 kube-scheduler.yaml
```
We can see there are 4 static pods here, since pod def are present inside this manifest dir. 

On which nodes are the static pods created currently?
```
root@controlplane:~# kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   12m   v1.20.0
node01         Ready    <none>                 10m   v1.20.0

root@controlplane:~# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
kube-system   coredns-74ff55c5b-bn2l8                1/1     Running   0          11m   10.244.0.3    controlplane   <none>           <none>
kube-system   coredns-74ff55c5b-t8n74                1/1     Running   0          11m   10.244.0.2    controlplane   <none>           <none>
kube-system   etcd-controlplane                      1/1     Running   0          11m   10.18.91.9    controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane            1/1     Running   0          11m   10.18.91.9    controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane   1/1     Running   0          11m   10.18.91.9    controlplane   <none>           <none>
kube-system   kube-flannel-ds-s799x                  1/1     Running   0          11m   10.18.91.9    controlplane   <none>           <none>
kube-system   kube-flannel-ds-stskc                  1/1     Running   0          11m   10.18.91.12   node01         <none>           <none>
kube-system   kube-proxy-fl68w                       1/1     Running   0          11m   10.18.91.12   node01         <none>           <none>
kube-system   kube-proxy-sscnq                       1/1     Running   0          11m   10.18.91.9    controlplane   <none>           <none>
kube-system   kube-scheduler-controlplane            1/1     Running   0          11m   10.18.91.9    controlplane   <none>           <none>
root@controlplane:~# 
```
Create a static pod named static-busybox that uses the busybox image and the command sleep 1000,
```
root@controlplane:~# kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

root@controlplane:~# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE    IP            NODE           NOMINATED NODE   READINESS GATES
default       static-busybox-controlplane            1/1     Running   0          118s   10.244.0.4    controlplane   <none>           <none>
```
How to find the manifest or static pod dir/path?

Look for the pod yaml in dir `/etc/kubernetes/manifests/` if pod definition is not there then do,
```
root@node01:/etc/kubernetes/manifests# ps aux | grep kubelet
root     12447  0.0  0.0 3780872 96440 ?       Ssl  14:18   0:08 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
```
From here we got to know the `--config=/var/lib/kubelet/config.yaml`, 
```
root@node01:~# cat /var/lib/kubelet/config.yaml | grep staticPodPath:
staticPodPath: /etc/just-to-mess-with-you
```
Thats It!

### Multiple Schedulers

If the default scheduler does not suit your needs you can implement your own scheduler. ... Moreover, you can even run multiple schedulers simultaneously alongside the default scheduler and instruct Kubernetes what scheduler to use for each of your pods.

```
Default scheduler:

root@controlplane:~# kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
kube-scheduler-controlplane            1/1     Running   0          7m9s

/etc/kubernetes/manifests/kube-scheduler.yaml
```

Creating a pod with your custom scheduler,
```
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx 
spec:
  schedulerName: my-scheduler
  containers:
  - image: nginx
    name: nginx
    
    
root@controlplane:~# kubectl explain pod --recursive | less
root@controlplane:~# vim /root/nginx-pod.yaml 
root@controlplane:~# kubectl create -f nginx-pod.yaml 
pod/nginx created
```

### Metric Server

* To monitor kubernetes each of nodes and pods
* This is in-memory monitoring solution and doesnt store data on disk
* How are metric generated for pods in this nodes? Kubernetes runs an agent on each node (kubelet). Thisalso contains c-advisor (retreving performance data). 

```
root@controlplane:~/kubernetes-metrics-server# kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   302m         0%     963Mi           0%        
node01         204m         0%     687Mi           0%   

root@controlplane:~/kubernetes-metrics-server# kubectl top pods
NAME       CPU(cores)   MEMORY(bytes)   
elephant   37m          52Mi            
lion       2m           7Mi             
rabbit     104m         203Mi  
```

Imp:

```
kubectl top pods -l app=myapp
```
-------------------------------------------------- DAY 10: --------------------------------------------------
### Managing application logs

```
root@controlplane:~# kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          29s
root@controlplane:~# kubectl logs -f webapp-1
```

Check logs,
```
root@controlplane:~# kubectl logs -f webapp-1
[2021-07-11 12:05:58,104] INFO in event-simulator: USER3 logged in
[2021-07-11 12:05:59,106] INFO in event-simulator: USER4 is viewing page2
[2021-07-11 12:06:00,107] INFO in event-simulator: USER3 is viewing page2
[2021-07-11 12:06:01,109] INFO in event-simulator: USER3 is viewing page1
[2021-07-11 12:06:02,110] INFO in event-simulator: USER4 logged out
[2021-07-11 12:06:03,112] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-07-11 12:06:03,112] INFO in event-simulator: USER3 logged out
[2021-07-11 12:06:04,113] INFO in event-simulator: USER4 logged in
[2021-07-11 12:06:05,114] INFO in event-simulator: USER3 is viewing page2

root@controlplane:~# kubectl logs -f webapp-2 simple-webapp 
root@controlplane:~# kubectl logs -f webapp-2 -c simple-webapp 
```

### Rolling update & Rollback in deployment

Rollout & versioning in a deployment

* When you first create a deployment it triggers a rollout
* A new rollout create a new deployment revisons

There are two deployment strategy:

* Recreate pods (Application downtime)
* Rolling update (Default deployment strategy)


```
root@controlplane:~# 
root@controlplane:~# kubectl get deployment
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           3m1s

root@controlplane:~# kubectl describe deployments.apps frontend
Name:                   frontend
Namespace:              default
RollingUpdateStrategy:  25% max unavailable, 25% max surge.      // strategy default
```
If you were to upgrade the application now what would happen? >> PODs are upgraded few at a time

Update current deployment by modifying the image to version 2,
```
root@controlplane:~# kubectl describe deployments.apps frontend | grep Image 
    Image:        kodekloud/webapp-color:v1
    
root@controlplane:~# kubectl edit deployments.apps frontend 
deployment.apps/frontend edited

root@controlplane:~# kubectl describe deployments.apps frontend | grep Image 
    Image:        kodekloud/webapp-color:v2
```
### Commands & Arguments in Docker

![Screenshot 2021-07-11 at 9 26 20 PM](https://user-images.githubusercontent.com/29716063/125201928-d9aa9280-e28e-11eb-9053-f5b8afd486a2.png)

What is the command used to run the pod ubuntu-sleeper?
```
root@controlplane:~# kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
ubuntu-sleeper   1/1     Running   0          112s
root@controlplane:~# kubectl describe pod ubuntu-sleeper | grep -i command:
    Command:
       sleep
      4800
      
Ans:  Sleep 4800
```
Create a pod with the ubuntu image to run a container to sleep for 5000 seconds,
```
root@controlplane:~# cat ubuntu-sleeper-2.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
     - "sleep"
     - "5000"
     
root@controlplane:~#  kubectl create -f ubuntu-sleeper-2.yaml
pod/ubuntu-sleeper-2 created     
```
Inspect the file Dockerfile given at /root/webapp-color. What command is run at container startup? [python app.py]
```
root@controlplane:~/webapp-color# cat Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```
Inspect the file Dockerfile2 given at /root/webapp-color. What command is run at container startup? [python app.py --color red]
```
root@controlplane:~/webapp-color# cat Dockerfile2 
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```
Inspect the two files under directory webapp-color-2. What command is run at container startup? Assume the image was created from the Dockerfile in this folder.
```
root@controlplane:~/webapp-color-2# cat Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

root@controlplane:~/webapp-color-2# cat webapp-color-pod.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```
The ENTRYPOINT in the Dockerfile is overridden by the command in the pod definition.

Ex 2:

```
root@controlplane:~/webapp-color-3# cat Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

root@controlplane:~/webapp-color-3# cat webapp-color-pod-2.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["python", "app.py"]
    args: ["--color", "pink"]
```
Inspect the command and args in the pod definition file [python app.py --color pink]

-------------------------------------------------- DAY 11: --------------------------------------------------

