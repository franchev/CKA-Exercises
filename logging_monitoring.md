<h1>Logging/Monitoring 5%</h1>

<h2>Understand how to monitor all cluster components</h2>
<p>Kubernetes does not come pre-installed with metrics monitoring. </p>

<h3>Minikube enable metrics-server</h3>
  
<details><summary>Answer</summary>
<p>

```bash
minikube addons enable metrics-server
```

</p>
</details>
  

<h3>non-minikube environment, enable metrics-server</h3>

<details><summary>Answer</summary>
<p>

```bash
git clone https://github.com/kubernetes-incubator/metrics-server.git
kubectl create -f deploy/1.8+/
```

</p>
</details>


<h3>View cluster metric information after metrics-server is installed</h3>

<details><summary>Answer</summary>
<p>

```bash
kubectl top node  # View node CPU and memory usage
kubectl top pod   # View pod CPU and memory usage
```

</p>
</details>


<h2>Understand how to monitor applications</h2>

<h2>Manage cluster component logs</h2>
<h3>View all nodes in your cluster</h3>
<details><summary>Answer</summary>
<p>

```bash
kubectl get nodes -o wide
```

</p>
</details>

<h3> Looking at logs on the master</h3>
<details><summary>Answer</summary>
<p>

```bash
ssh master1  #ssh to the master node
less /var/log/kube-apiserver.log  # API Server, responsible for serving the API
less /var/log/kube-scheduler.log  # Scheduler, responsible for making scheduling decisions
less /var/log/kube-controller-manager.log # Controller that manages replication controller
```

</p>
</details>


<h3> Looking at logs on worker nodes</h3>
<details><summary>Answer</summary>
<p>

```bash
ssh node1   # ssh to a worker node
less /var/log/kubelet.log # kubelet, responsible for running containers on the node
less /var/log/kube-proxy.log # kube proxy, responsible for service load balancing
```

</p>
</details>



<h2> Manage application logs</h2>
<h3>Create a pod using the image:busybox that runs this command 'i=0;while true;do echo "hello number $i"; i=$((i+1));sleep 1; done'. View its log</h3>
  
<details><summary>Answer</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh 'i=0;while true;do echo "hello number $i"; i=$((i+1));sleep 1; done'
kubectl logs busybox -f # similar to tail -f stream log live
kubectl logs busybox # prints the last logs from the pod
```

</p>
</details>
