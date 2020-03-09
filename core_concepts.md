<h1>Core Concepts 19%<h1>

<h2>Understand the Kubernetes API primitives</h2>

<h2>Understand the Kubernetes cluster architecture.</h2>
<p> The cluster is made of the Master & worker nodes</p>
<p> The master node is responsible for managing, planning, scheduling, & monitor nodes. Here are the components that lives on the master node(s) </p>
<li> ETCD Cluster </li>
<li> Kube-apiserver </li>
<li> Kube Controller Manager </li>
<li> Kube-scheduler </li>

<p> The worker node is responsible for hosting application as containers. Here are the components that ives on the worker node(s)<p>
  <li> kubelet agent </li>
  <li> kube-proxy </li>
  <li> Container Runtime Engine (also on master node) </li>

<p> There are many different component in a Kubernetes Cluster & they all have their own purpose </p>
<p> Cluster Components </p>
<li> ETCD Cluster: Highly available key value store. A database that stores information in a key value base format. </li>
<li>Kube-scheduler: Identifies the right node to place a container on based on the containers resource requirements, worker nodes capacity, or based on any other policies or constrainst such as taints or node affinity rules</li>
<li>Kube-Controller-Manager: When you install kubernetes controller manager the different controllers get installed. There are many controllers that Kubernetes uses to control the different part of the cluster.  </li>
<li>Node-Controller: Takes care of nodes, they are responsible for unboarding new nodes to the cluster.</li>
<li>Replication-Controller: Ensures that the desired number of containers are running at all times</li>
<li>Kube-apiserver: The primary management component of Kubernetes. It is responsible for orchastrating all operations within the cluster. It exposes the Kubernetes API which is used by external users to perform management operations on the cluster as well ast he various controllers to monitor the state of the cluster.   </li>
<li>Container Runtime Engine: a tool to run containers for example Docker, ContainerD or Rocket. This needs to be installed on all the nodes in the cluster including the master nodes.</li>
<li>Kubelet: An agent that runs on each node in the Cluster. It listens for instructions from the kube-api server and deploys or destroys containers on the nodes. </li>
<li>Kube-proxy: This runs on the worker nodes. It ensures that the necessary rules are in place on the worker nodes to allow the containers running on them to reach each other. </li>
</ul>


<h3> Install ETCD On linux</h3>

<details><summary>Answer</summary>

```bash
# Download binary
curl -L https://github.com/etc
# Extract binary
tar xzvf etcd-v3***.tar.gz
# Run it
./etcd
```
</details>


<h3> Create a namespace named dev-namespace, then create a pod named nginx with image nginx in the dev-namespace</h3>

<details><summary>Answer</summary>

```bash
kubectl create namespace dev-namespace
kubectl run nginx --image=nginx --restart=Never -n dev-namespace
```
</details>

<h3> Create a pod named nginx with image nginx in the dev-namespace using a pod definition file</h3>

<details><summary>Answer</summary>

```bash
kubectl run gninx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yaml

vim nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f nginx-pod.yaml -n dev-namespace

OR
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml | kubectl create -n dev-namespace -f -
```
</details>


<h3> Create Replication Controller using a yaml file</h3>

<details><summary>Answer</summary>

```bash
vi rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: frontend
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 2

kubectl apply -f rc.yaml

```
</details>


<h3> Create a replicaSet using yaml</h3>

<details><summary>Answer</summary>

```bash
vi rset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rset
  labels:
    app: myapp
    type: frontend
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 2
  selector: 
    matchLabels:
      type: frontend

kubectl apply -f rset.yaml

kubectl get replicaset    -> to view replicasets
```
</details>


<h3> Scale replicaset myapp-rset to 6</h3>

<details><summary>Answer</summary>

```bash
kubectl scale --replicas=6 replicaset myapp-rset
```
</details>

<h3> Create a deployment with image nginx. Name the deployment nginx. Set it to have 2 replicas. Expose port 80 without using a service </h3>

<details><summary>Answer</summary>

```bash
kubectl run nginx --image=nginx --replicas=2 --port=80

alternative (new approach)
kubectl create deployment nginx --image=nginx --dry-run -o yaml > deploy.yaml

# then edit deploy.yaml
# increase replicas to 2
# in the containers section add the ports value as below
vi deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
          - containerPort: 80
        resources: {}
status: {}
```
</details>





</details>

<h3>Get pods on all namespaces</h3> 

<details><summary>Answer</summary>

```bash
kubectl get pod --all-namespaces
```
</details>

<h2>Understand Services and other network primitives</h2>

<p> Kubernetes services enable communication between various components within and outside of the application. Services help us connect applications together with other applications or users.<p>

<p> Services Types </p>
<li> NodePort: the service makes an internal POD accessible on a port on the Node </li>
<li> Cluster IP: the service creates a virtual IP inside the cluster to enable communication between different services such as a set of front end servers to a set of back end servers </li>
<li> LoadBalancer: the service provisions a load balancer for our service in supported cloud providers</li>

<h3>Create a nodeport service named myapp-service pointing to nods with label type: front-end and app: myapp. Ensure that the service targetport is set to 80 & the nodeport value is set to 30008</h3> 

<details><summary>Answer</summary>

```bash
vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end

kubectl apply -f svc.yaml

(alternative approach)
# generate the yaml file using the create service nodeport command
# edit the file and change the selector section to include the 2 labels type & frontend
# apply the yaml file
kubectl create service nodeport myapp-service --tcp=80:80 --node-port=30008 --dry-run -o yaml> svc.yaml
kubectl apply -f svc.yaml
```
</details>

<h3>Create a deployment with the following attributes: image=nginx, replicas=3, pod label (app: myapp, type: frontend)</h3> 

<details><summary>Answer</summary>

```bash
```
</details>

<h3>Create another deployment with the following attributes: image=busybox, replicas=3, pod label (app: my-backend, type: mysql</h3> 

<details><summary>Answer</summary>

```bash
```
</details>

<h3>Create a service named redis-service of type ClusterIP that expose pod redis on port 6379</h3> 

<details><summary>Answer</summary>

```bash
kubectl expose pod redis --port=6379 --name redis-service  -o yaml

# alternative generate yaml file
# then edit the yaml file and change the selector to point to the label of the pod
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml > svc.yaml

```
</details>