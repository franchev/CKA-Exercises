<h1>Cluster 11%</h1>

<h2>Understand Kubernetes cluster upgrade process</h2>

<h3> View current version of kubernetes installed on your nodes </h3>

<details><summary>Answer</summary>

```bash
kubectl get nodes       # this will print the current version of kubernetes in the cluster for each node
```

</details>


<p>When dealing with upgrades, remember that the kube-apiserver has to have the highest version. No other component should have a higher version than the kube-apiserver (exception, the kubectl is allowed to be higher)</p>


<h3> Upgrade admin-1 using kubeadm</h3>

<details><summary>Answer</summary>

```bash
kubeadm upgrade plan    # this will list the current versions of the different components, and which version is available to upgrade to.
apt-get upgrade -y kubeadm=1.12.0-00  # upgrades the kubeadm tool to version 1.12.0
kubecadm upgrade apply v1.12.0        # upgrades the cluster to version v1.12.0

apt-get upgrade -y kubelet=1.12.0-00   # you must upgrade the kubelet component manually on each node/master (if you have it installed on the master node)
systemctl restart kubelet

kubectl get nodes                      # view which version is installed

```

</details>

<h3> Upgrade node-1 using kubeadm</h3>

<details><summary>Answer</summary>

```bash
kubectl drain node-1    # terminate pods & recreated them in other nodes (if applicable). Mark node-1 as unschedulable. 

apt-get upgrade -y kubeadm=1.12.0-00  # upgrades the kubeadm tool to version 1.12.0
apt-get upgrade -y kubelet=1.12.0-00   # you must upgrade the kubelet component manually on each node/master (if you have it installed on the master node)
kubeadm upgrade node config --kubelet-version v1.12.0
systemctl restart kubelet
kubectl get nodes                      # view which version is installed
kubectl uncordon node-1                # make node schedulable again (able to create pods on this node, old pods will not necessarily be recreated on this node)

```

</details>


<h2>Facilitate operating system upgrades</h2>

<p>By default the kubernetes manager waits for 5 minutes for a down node to be deemed down. At this point, once a node is deemed down, the pods are evicted from that node. If the nodes are part of a replicaSet or a deployment, then they are recreated on other nodes. This eviction timeout is set by this command.
```bash
kube-contoller-manager --pod-eviction-timout=5m0s
```
</p>

<h3> Drain node-1 purposely to get it ready for OS upgrade</h3>

<details><summary>Answer</summary>

```bash
kubectl drain node-1    # pods get terminated on node-1, then created on other nodes. At this point, this node is unschedulable (meaning, when the pod is back only, new pods will not be created on it)

```

</details>

<h3>Make node-2 unschedulable without draining</h3>

<details><summary>Answer</summary>

```bash
kubectl cordon node-2   # node-2 will not accept new pod creation
```

</details>

<h3>Make node-1 schedulable after OS upgrade</h3>

<details><summary>Answer</summary>

```bash
kubectl uncordon node-1
```

</details>

<h2>implement backup and restore methodologies</h2>

<h3> backup all resources in your cluster to a file named all-deploy-services.yaml</h3>

<details><summary>Answer</summary>

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

</details>

<h3>Create a snapshot for the ETCD component</h3>

<details><summary>Answer</summary>

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db

ETCDTL_API=3 etcdctl snapshot status snapshot.db      # view the snapshot status

```

</details>

<h3>Restore a snapshot for the ETCD component</h3>

<details><summary>Answer</summary>

```bash
service kube-apiserver stop                          # stop the kube-apiserver component

ETCDTL_API=3 etcdctl \
snapshot restore snapshot.db \
--data-dir /var/lib/etcd-from-backup \
--initial-cluster master-1=https//:[ip]:2380,master-2=https://[ip]:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-advertise-peer-urls https://${INTERNAL_IP}:2380

vi /etc/systemd/system/etcd.service
change these 2 parameters to point to the new location:
--intial-cluster-token
--data-dir

systemctl daemon-reload
service etcd restart
service kube-apiserver start


```

</details>