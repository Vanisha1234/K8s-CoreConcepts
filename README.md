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
To check kube apiserver as pod - kubectl get pods -n kube-system
The pod definition file is located at - /etc/kubernetes/manifests/kube-controller-manager.yaml
Incase of non kubeadm setup - etc/systemd/system/kube-controller-manager.service
To check for running processes - ps aux | grep kube-controller-manager

KUBE SCHEDULER
Only responsible for deciding - Which pod goes on which node. Does not actually place the pod on a node.
Decides which node has sufficient capacity to accomodate the following pods. 
Depends upon certain criteria like , scheduler prioritizes the node which will be left with more number of resources even after scheduling the pod.
Other criteria include- Resource requirements and limits, Taints and toleration, Node selectors and Affinity.
To check kube apiserver as pod - kubectl get pods -n kube-system
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

















