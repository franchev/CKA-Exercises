<h1>Logging/Monitoring 5%</h1>

<h2>Understand how to monitor all cluster components</h2>
Kubernetes does not come pre-installed with metrics monitoring. 

<h4>Minikube enable metrics-server<h4>
  
<details><summary>Answer</summary>
<p>

```bash
minikube addons enable metrics-server
```

</p>
</details>
  

<h4>non-minikube environment, enable metrics-server</h4>

<details><summary>Answer</summary>
<p>

```bash
git clone https://github.com/kubernetes-incubator/metrics-server.git
kubectl create -f deploy/1.8+/
```

</p>
</details>


<h4>View cluster metric information after metrics-server is installed</h4>

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

<h2>Manage application logs</h2>

<h4>Create a pod using the image:busybox that runs this command 'i=0;while true;do echo "hello number $i"; i=$((i+1));sleep 1; done'. View its log<h4>
  
<details><summary>Answer</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh 'i=0;while true;do echo "hello number $i"; i=$((i+1));sleep 1; done'
kubectl logs busybox -f # similar to tail -f stream log live
kubectl logs busybox # prints the last logs from the pod
```

</p>
</details>
