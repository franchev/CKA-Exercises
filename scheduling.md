<h1>Scheduling 5%</h1>


<h2>Use label selectors to schedule Pods</h2>

<h3> Create a pod running nginx with lables (app=App1, function=web-server)</h3>

<details><summary>Answer</summary>

```bash
kubectl run nginx --image=nginx --restart=Never --labels=app=App1,function=web-server
kubectl get pods --selector app=App1    # to view pods created with label app=App1
```

</details>


<h3> Create a replicaSet that targets pods labeled app:App1 & function: web-server </h3>

<details><summary>Answer</summary>

```bash
vi replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset1
  labels:
    app: App1
    function: web-server-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: web-server
    spec:
      containers:
      - name: nginx
        image: nginx

The selector matchLabels will target existing pods. 

kubectl apply -f replicaset.yaml

```

</details>

<h2>Understand the role of DaemonSets</h2>

<p>Daemon Sets runs one copy of your pod in each node in your cluster. If you add a new node to the cluster, it'll automatically have a copy of that pod. Good case scenario is a logging pod or kubeproxy & networking. DaemonSets uses the default scheduler & Node Affinity to ensure that a pod is created on each node</p>


<h3> Create a DaemonSet monitoring agent. Use image monitoring-agent for your pod</h3>

<details><summary>Answer</summary>

```bash
vi daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent

kubectl apply -f daemonset.yaml
kubectl get daemonset  # to view daemonset created

```

</details>

<h2>Understand How resource limits can affect Pod Scheduling.</h2>

<p>The default value for CPU=0.5 and Mem=256Mi. If you don't specify, kubernetes will set your containers default resources requests to these values. You can go as low as 1m for CPU. </p>



<h3> Create a pod named webapp-color, image=nginx with a resource requests memory value of 1Gi and CPU=1. Also set the limits to cpu=2 & memory= 2Gi</h3>

<details><summary>Answer</summary>

```bash
vi pod-limits.yaml
apiVersion:
kind:
metadata:
  name: webapp-color
spec:
  containers:
  - name: webapp-color
    image: nginx
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
    limits:
      memory: "2Gi"
      cpu: 2

kubectl apply -f pod-limits.yaml
kubectl get pods                # list pods

```

</details>


<h2>Understand how to run multiple schedulers and how to configure Pods to use them</h2>

<h3> Deploy additional Scheduler</h3>

<details><summary>Answer</summary>

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler

create your custom scheduler.service
vi my-custom-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler \\
   --config=/etc/kubernetes/config/kube-scheduler.yaml \\
   --scheduler-name= my-custom-scheduler
```

</details>


<h3> Deploy additional Scheduler using kubeadm</h3>

<details><summary>Answer</summary>

```bash
cp /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes/manifests/my-custom-scheduler.yaml   # copying existing scheduler
vi my-custom-scheduler.yaml
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: my-custom-scheduler

kubectl apply -f my-custom-scheduler.yaml
```

</details>


<h3> View Schedulers </h3>

<details><summary>Answer</summary>

```bash
kubectl get pods --namespace=kube-system
```

</details>

<h3> Use my-custom-scheduler in a pod definition</h3>

<details><summary>Answer</summary>

```bash
vi pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-custom-scheduler

```

</details>






<h2>Manually schedule a pod without a scheduler</h2>


<h3> manually Schedule a pod name nginx with image nginx on node02</h3>

<details><summary>Answer</summary>

```bash

vi nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: node02

kubectl apply -f nginx.yaml

```

</details>

<h3> Bind an existing running pod to node03</h3>

<p>If you already have a running pod, you cannot edit it to change it's nodeName, instead you have to create a binding object</p>

<details><summary>Answer</summary>

```bash

vi pod-bind.yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: None
  name: node03

  Convert this yaml file to json, and use it to call the api
  curl --header "Content-Type:application/json" --request POST -- data '{"apiVersion":"v1", "kind":"Binding"... } \
  http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/

```

</details>




<h2>Display scheduler events</h2>

<h3>View events taking place </h3>

<details><summary>Answer</summary>

```bash
kubectl get events    # view events being done in the cluster
```

</details>


<h3> View logs for a scheduler named my-custom-scheduler</h3>

<details><summary>Answer</summary>

```bash
kubectl logs my-custom-scheduler --namespace=kube-system
```

</details>

<h2>Know how to configure the Kubernetes Scheduler</h2>