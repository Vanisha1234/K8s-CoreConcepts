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

In case of external communication
If we need to communicate with the application hosted on a pod, from outside the K8s cluster, we can use services insted on logging into the Node and then accessing.
Since usecase of services include- listening on the port of the node and then forwarding requests to the port on the pod hosting the application. This type of service is known as node port service.
Target port - The port on the pod; where the service forwards the request to.
Port - The port on the service itself; the service has a virtual IP assigned to it known as cluster IP.
NodePort - The port on the node. Node port has a range : 30000 - 32767

Service Definition File of type NodePort- svc.yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort #type of service
  ports:
    - targetPort: 80 #if targetPort is not provided, it will automatically pick up the same port of the port.
      port: 80 #this is important to provide
      nodePort: 30008 #if not provide, will be automatically set to any port between the valid range
  selector: #labels and selectors will be used to link the service to the pod to which the request is to be forwarded
      app: myapp
      type: frontend

To create a service - kubectl create -f svc.yml
To list the services - kubectl get services
To describe a service - kubectl describe service serviceName
We can now browse the application using the IP of the Node and the Port on the Node.

Cluster Ip type of service- Virtual Ip created inside the cluster used to enable communication between different services like set of frontend servers communicating to set of backend services
In case of various services running for eg set of frontend webservers and set of backen webservers; both needs to commnicate with each other but ips of pods are not static and may change if a pod is replaced hence it is not reliable for internal communication.
Secondly it becomes difficult to decide which pod instance to forward traffic to when all are same.
Hence this service helps in grouping the same pods together and provide a single interface to communicate.
Each service is assigned a name and the ip which will be used by pods for communication.

Service definition file of type Cluster IP- svc.yml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP #if type of service is not specified;this is the default svc type
  ports:
    - targetPort: 80 #port where backend is exposed
      port: 80 
  selector: #to link svc to backend pods
      app: myapp
      type: backend

The svc can be accessed by the pods using the clusterIP or the service name.

Load balancer type of service- Helps to distribute load across various webservers
In case of multiple pod instances running of the same application , each pod has same labels which is used in Load balancer service to link all the pods with same labels.
Once the pods are linked using labels, the service then automatically use all the pods as the endpoints to forward the request.
This service acts as a built in loadbalancer to distribute load.

In case of pods present in different nodes inside the cluster; no additional configuration is required; k8s automatically spans across the nodes in the cluster and map the target port to the same nodePort on all the nodes in the cluster. This way the application can be access through any ip and the same port.
Services are automatically updated if pos are added or removed.

NAMESPACES
An isolated environment that contains all the services like pods, svcs and deployments. There can be various namespaces like dev, prod and test. To isolate these environments namespaces are created so that users dont end up accidently making changes to another environment.
The resources within the namespace can refer to each other by their names.
To connect to service in the another namespace for eg - connection web pod to db pod in another namespace; we need to use - servicename.namespace.svc.cluster.local
To list the pods in a specific namespace - kubectl get pods --namespace=kube-system
To create a pod in a specific namespace - kubectl create -f pod.yml --namespace=blue-namespace
To avoid using namespaces in commands repeatedly, define namespace under metadata of the pod definition file.

Namespace definition file- ns.yml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

To create using definition file - kubectl create -f ns.yml
To create directly without creating def. file - kubectl create namespace dev
To avoid specifying ns repeatedly while running commands use - kubectl config set-context $(kubectl config current-context) --namespace=dev
To view pod in all the namespaces - kubectl get pods --all-namespaces

To limit resources of a namespace like cpu or memory usage; create a resource quota
Resource Quota definition file - rq.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10 Gi
    
 To create resource quota via definition file - kubectl create -f rq.yaml     
  
IMPERATIVE & DECLARATIVE
Imperative Approach means specifying what and how to do.
Declarative Approach means specifying what to do instead of specifying the details. The rest is handled by the system itself.

Example of Imperative Approach
Imperative commands run once and require long commands in case of complex environments. They are limited to user who ran these commands. Difficult to keep track of.
To create a pod - kubectl run --image=nginx nginx                                               |
To create a deployment - kubectl create deployment --image=nginx nginx                          |  --------> To create an object
To create a service to expose deployment - kubectl expose deployment nginx --port 80            |

To edit the existing object - kubectl edit deployment nginx                                     |
To scale a deployment - kubectl scale deployment nginx --replicas=5                             |  ---------> To update an object
to update an image on deployment - kubectl set image deployment nginx nginx=nginx:1.18          |
To create an object specifying file - kubectl create -f nginx.yaml
To edit an object specifying file - kubectl replace -f nginx.yaml
To delete an object specifying file - kubectl delete -f nginx.yaml

To create , update and delete an object - kubectl apply -f nginx.yaml
We create yaml files, which are easy to keep trackof, easy to share and update.
Ways of editing the existing object
1. if used - kubectl edit deployment nginx
This command will open similar yaml file that was used to create an object but with additional fields - This is not the file used to create an object. This is similar yaml file in k8s memory and changes made to it will be applied on live.
However, there is a difference between local yaml file and the k8s memory yaml file-
The changes applied to k8s memory yaml file are not recorded anywhere and once the changes are made, we are left with local file which has no changes in it.
2. Better approach is to make changes to local yaml file and then run - kubectl replace -f nginx.yaml
3. To completely delete of recreate an object - kubectl replace --force -f nginx.yaml


Examples of Declarative Approach
In case of Declarative approach we use the same yaml files that was used in imperative approach but in-place of create or replace commands we use - kubectl apply -f nginx.yaml; to manage the object
Kubectl apply command is intelligent enough to create an object if it doesnt exist.
To create multiple objects, the objects can be specifies in a directory and the directory path can be provided to the apply command - kubectl apply -f /path/to/directory
To update the existing fie, changes are to be made in congif file and the command to be run is - kubectl apply -f nginx.yaml

To create a pod - kubectl run nginx --image=nginx ; Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run) -  kubectl run nginx --image=nginx --dry-run=client -o yaml
To create a deployment - kubectl create deployment --image=nginx nginx ; Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) - kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
To save the YAML definition file generated to a file and modify - kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
Generate Deployment with 4 Replicas - kubectl create deployment nginx --image=nginx --replicas=4
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379 - kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:] - kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml

KUBECTL COMMAND
To list all resources - kubectl api-resources ; list names , shortnames, apiVersions and other details about all the resources.
to explain an object, run - kubectl explain pod ; For details - kubectl explain pod --recursive

