<h1>Core Concepts 19%<h1>

<h2>Understand the Kubernetes API primitives</h2>

<h2>Understand the Kubernetes cluster architecture.</h2>
<p> The cluster is made of the Master & worker nodes</p>
<p> The master node is responsible for managing, planning, scheduling, & monitor nodes. Here are the components that lives on the master node(s) </p>
* ETCD Cluster
* Kube-apiserver
* Kube Controller Manager
* Kube-scheduler

<p> The worker node is responsible for hosting application as containers. Here are the components that ives on the worker node(s)<p>
* kubelet agent
* kube-proxy
* Container Runtime Engine (also on master node)

<p> There are many different component in a Kubernetes Cluster & they all have their own purpose </p>
<ul> Cluster Components
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

<h3> </h3>

<details><summary>Answer</summary>

```bash
```
</details>


</details>

<h3> </h3>

<details><summary>Answer</summary>

```bash
```
</details>

<h2>Understand Services and other network primitives</h2>