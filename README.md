# K8s-CoreConcepts
ETCD
Distributed, reliable, key value store
(key value store - stores a value to a key for the given data. Fast & Highly flexible. It has no schema, no complex queries)
Listens on 2379 by default
Either will be running as a system service or a pod
To check etcd as pod - kubectl get pods -n kube-system
etcd client - Any client attcahed to etcd server to store and retrieve information. Default client name - etcd ctl client
Role in Kubernetes- store data regarding the cluster like - 
Nodes, Pods, Configs, Secrets, Accounts, Roles, Bindings, others
Everything is updated in etcd server and every response is from etcd server. Change is considered complete if it is updated in the etcd server.

KUBE-APISERVER
Primary Management Component in K8s
All requests and responses goes through Kube apiserver
Kube apiserver is the only component that communicates with etcd
Responsible for - Authenticate User, Validate Requests, Retrieve data, Update ETCD, used by scheduler and kubelet to update data in the cluster.
To check kube apiserver as pod - kubectl get pods -n kube-system
The pod definition file is located at - /etc/kubernetes/manifests/kube-apiserver.yaml
Incase of non kubeadm setup - etc/systemd/system/kube-apiserver.service
To check for running processes - ps aux | grep kube-apiserver

KUBE CONTROLLER MANAGER
Its a process that continuously monitors the state of various components in a cluster and work towards bring the system to the desired functioning state.
Eg - 
Node Controller - It is responsible for monitoring the status of the nodes and take necessary actions to keep the application running. It recieves node status via Kube apiserver. If there is no response from the node for more than 40 seconds, it marks it as unreachable and in case the node in unreachable for more than 5 mins ,it replaces it.

Replication Controller - Reponsible for monitoring the status of the replicasets and ensure desired number of pods are available all time within the set.
There are various controllers which are packaged inside a single process known as - Kubernetes Controller Manager
To check kube controller as pod - kubectl get pods -n kube-system
The pod definition file is located at - /etc/kubernetes/manifests/kube-controller-manager.yaml
Incase of non kubeadm setup - etc/systemd/system/kube-controller-manager.service
To check for running processes - ps aux | grep kube-controller-manager

KUBE SCHEDULER
Only responsible for deciding - Which pod goes on which node. Does not actually place the pod on a node.
Decides which node has sufficient capacity to accomodate the following pods. 
Depends upon certain criteria like , scheduler prioritizes the node which will be left with more number of resources even after scheduling the pod.
Other criteria include- Resource requirements and limits, Taints and toleration, Node selectors and Affinity.
To check kube scheduler as pod - kubectl get pods -n kube-system
The pod definition file is located at - /etc/kubernetes/manifests/kube-scheduler.yaml
Incase of non kubeadm setup - etc/systemd/system/kube-scheduler.service
To check for running processes - ps aux | grep kube-scheduler


KUBELET
Sole point of contact between the worker node and the master node
Reponsible for-
loading and unloading containers
send reports of containers status
kubelet in worker node registers the node with kubernetes cluster
It recieves instruction to load a container/pod on a worker node which later requests the container runtime present on the worker node to pull image and run container.
Monitors nodes and pods and sends report/status to Kube Apiserver timely basis.
Kubelet is always installed manually and not directly from binaries.
To check for running processes - ps aux | grep kubelet

KUBE PROXY
In order to make services accessible throughout the cluster , kube proxy is used and is installed on every node.
It is Responsible to create rules on each pod everytime a new service is created  to forward the traffic to that service.
It does this using iptables rules.
To check kubeproxy as pod - kubectl get pods -n kube-system

PODS
Containers are not deployed directly in kubernetes. They are encapsulated in a kubernetes objects called pods.
Its a single instance of an application and smallest object in a k8s cluster.
If load increases, a new pod with a container is deployed / a new node with a new pod is deployed within the cluster. 
Scaling up will spin more pods and scaling down with remove pods.
A pod can contains two containers of different types but not of similar types.
With pod we are not required to set up any thing and it establishes all the network connectivity itself when a seperate container is created inside a pod and hence pods is considered even with a single container so that its easy to scale in case of future architectural changes.
To deploy a pod - kubectl run pod nginx --image nginx
List pod - kubectl get pods

K8s Pod Definition File - pod.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: myapp
  labels:
      app: myapp-pod
      type: frontend
spec:
  containers:
    - name: nginx-cont
      image: nginx
To create a new pod definition file - kubectl create -f pod.yaml / kubectl apply -f pod.yaml 
To get detail info of a pod - kubectl describe pod podname
To delete a pod - kubectl delete pod podname
To edit a pod - kubectl edit pod pod name ; Apply changes - kubectl apply -f pod.yaml
To edit a pod created via vi, vim or nano tool - vim podyamlfile ; Apply changes - kubectl apply -f pod.yaml

REPLICATION CONTROLLER & REPLICASETS
Helps to prevent downtime by maintaining the desired number of pods at all times for high availability.
It can also replace an existing unhealthy pod if the desired number of pod is just 1.
It is also responsible for load balancing and scaling across multiple pods as well as multiple nodes in case of increasing load.

Replication-controller definition file - rc.yaml
apiVersion: v1 
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
      app: myapp
      type: frontend
spec:
  template: #template block to define the pod definition file for which replications will be created.
     metadata:
       name: myapp
       labels:
          app: myapp-pod
          type: frontend
     spec:
       containers:
         - name: nginx-cont
           image: nginx
  replicas: 3 #number of replicas

To create a replication controller definition file - kubectl create -f rc.yaml
To list replicasets - kubectl get replicationcontroller

ReplicaSet definition file - rs.yaml
apiVersion: apps/v1 
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
      app: myapp
      type: frontend
spec:
  template: #template block to define the pod definition file for which replications will be created.
     metadata:
       name: myapp
       labels:
          app: myapp-pod
          type: frontend
     spec:
       containers:
         - name: nginx-cont
           image: nginx
  replicas: 3 #number of replicas
  selector: #helps replicaset identify what pods falls under it
     matchLabels:
        type: frontend
To update a replicaset - kubectl scale --replicas=6 rs.yaml
Another way is to update the yaml file and run - kubectl replace -f rs.yaml ; to replace the replicaset

DEPLOYMENT
Responsible for:
Deploying multiple instances of webserver
To update multiple docker instances seamlessly using rolling update strategy
To carry out easy rollbacks incase of bugs and issues seamlessly altogether.
To make changes to application running on multiple docker instances

Pod help us deploying the single instance of our application; replicaset help us deploy mutliple pods; deployment allow us to upgrade the underlying instances seamlessly
Deployment definition file - (Similar to replicaset just the kind changes)
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: myapp-replicaset
  labels:
      app: myapp
      type: frontend
spec:
  template: 
     metadata:
       name: myapp
       labels:
          app: myapp-pod
          type: frontend
     spec:
       containers:
         - name: nginx-cont
           image: nginx
  replicas: 3 #number of replicas
  selector: 
     matchLabels:
        type: frontend

To create a deployment - kubectl create -f deployment.yaml
To list deployments - kubectl get deployments
Deployment automatically creates a replicaset hence run - kubectl get replicaset; to list replicaset created
Replicaset automatically created pods hence run - kubectl get pods; to list the running pods
To list all the created objects at one run - kubectl get all 

SERVICES
Enables Communication between various components within and outside the application; like connects frontend with backend pods
Helps connect different applications and users
Services enabling loose coupling between microservices in our application






