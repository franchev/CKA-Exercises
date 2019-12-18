<h1> Application Lifecylce Management 8%</h1>

<h2>Understand Deployments and how to perform rolling updates and rollbacks</h2>

<p>There are 2 types of deployment strategy</p>

<li>Recreate Strategy: not the default strategy, because this involves the deletion of all pods prior to recreating them. This means that your app is down as it's beeing updated</li>
<li>Rolling Update Strategy: This is the default strategy. This involves taking one pod down at a time, so that the application is working while being upgraded</li>

<h3> Create a deployment named nginx running image: nginx:1.7.1</h3>

<details><summary>Answer</summary>

```bash
kubectl run nginx image=nginx:1.7.1
kubectl get deploy   # to see the list of deployments and their status
```

</details>

<h3> Change the image of the deployment created above to be nginx:1.8.1</h3>

<details><summary>Answer</summary>

```bash
kubectl set image deployment/nginx nginx=nginx:1.8.1
```

</details>

<h3> Rollback to the previous deployment version with image: nginx:1.7.1</h3>

<details><summary>Answer</summary>

```bash
kubectl rollout undo deployment/nginx
kubectl rollout status deployment/myapp     # allows you to view the status of the update
kubectl rollout history deployment/myapp    # gives you a list of past updates
```

</details>

<h3> Rollback to the 2nd version of the deployment  with image: nginx:1.8.1</h3>

<details><summary>Answer</summary>

```bash
kubectl rollout undo deployment/nginx --revision=2
```

</details>




<h2>Know various ways to configure applications</h2>

<h3> Create a pod named sleeper with image: ubuntu-sleeper pass an argument value of 10</h3>

<details><summary>Answer</summary>

```bash
kubectl run sleeper --image=ubuntu-sleeper --restart=Never -- 10
```

</details>

<h3> Create a pod named hello with image: busybox that executes command echo 'hello world' </h3>

<details><summary>Answer</summary>

```bash
kubectl run hello --image=busybox --restart=Never --command echo 'hello world'
```

</details>

<h3> Create a configMap named app-data, with these two key value pair (username: admin, password:welcome). Create a pod named busybox, which uses image=busybox that passes that configMap as environment variables</h3>

<details><summary>Answer</summary>


```bash
kubectl create configmap app-data \
--from-literal=username=admin \
--from-literal=password=welcome

kubectl get configmaps     # to verify that the configMap was created

kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml > busybox.yaml        # generate pod definition file
vi busybox.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    envFrom:
    - configMapRef:
        name: app-data
  restartPolicy: Never

kubectl apply -f busybox.yaml # to create the pod

```

</details>

<h3> Create a configMap named app_config from a file named \tmp\app_config.properties. Create a pod with image=ubuntu that injects a single environment variable named: LIVES. Set the valueFrom the configMap of key=lives</h3>

<details><summary>Answer</summary>

```bash
cat app_config.properties
enemies=aliens
lives=3
allowed="true"

kubectl create configmap app_config \
--from-file=\tmp\app_config.properties

kubectl get configmaps   # verify that the configmap was created
kubectl describe configmap app_config # checkout the keys/values that are set

kubectl run ubuntu --image=ubuntu --restart=Never --dry-run -o yaml > ubuntu-pod.yaml    # generate pod definition

vi ubuntu-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ubuntu
  name: ubuntu
spec:
  containers:
  - image: ubuntu
    name: ubuntu
    env:
    - name: LIVES
      valueFrom:
        configMapKeyRef:
          name: app_config
          key: lives
  restartPolicy: Never

kubectl apply -f ubuntu-pod.yaml    # create pod

```

</details>

<h3> Create a configmap named app-data with key (app=web). Create a pod definition using image nginx. Inject the configmap named app-data as a volume in the pod</h3>

<details><summary>Answer</summary>

```bash
kubectl create configmap app-data \
--from-literal=app=web

kubectl get configmaps   # to verify that the configmap was created

kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yaml    # generate pod definition file

vi nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  volumes:
  - name: app-data-volume
    configMap:
      name: app-data
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: app-data-volume
      mountPath: /tmp/app
  restartPolicy: Never

kubectl apply -f nginx.yaml

kubectl exec -it nginx -- /bin/sh -c "ls /tmp/app"  # verify that the configmap is mounted as a volume.
```

</details>

<h3> </h3>

<details>Create a secret named mysql-secret configure it with these key/value data (DB_Host: mysql, DB_User: root, DB_Password: passwd). Create a pod named mysql using image mysql. Set the envFrom to point to the mysql-secret secret object <summary>Answer</summary>

```bash
kubectl create secret generic mysql-secret \
--from-literal=DB_Host=mysql \
--from-literal=DB_User=root \
--from-literal=DB_Password=passwd

kubectl get secrets # to get a list of secrets
kubectl describe secret mysql-secret # to view the content of the secret file. Secrets use base64 encoding. to decode use echo "" | base64 --decode

mysql-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
spec:
  containers:
  - name: mysql
    image: mysql
    envFrom:
      - secretRef:
          name: mysql-secret


kubectl apply -f mysql-pod.yaml
```

</details>


<h2>Know how to scale applications</h2>

<h3> Create a deployment named nginx with image=nginx. Set replicaSet to 3. After creation, modify the deployment and scale up replica value to 5. View history of changes to the deployment.</h3>

<details><summary>Answer</summary>

```bash

```

</details>

<h3> Use the nginx deployment above, and revert the deployment to revision 1</h3>

<details><summary>Answer</summary>

```bash

```

</details>

<h3> Modify the nginx deployment above, and add label (app=web-server)</h3>

<details><summary>Answer</summary>

```bash

```

</details>


<h3> Create a multi-container pod. use example with fluentd</h3>

<details><summary>Answer</summary>

```bash

```

</details>

<h3> Create a pod with image=busybox:1.28 & is named myapp. Set it to run command: "echo the app is active && sleep 3600". Add an initContainer that uses image busybox. Set its name to be init-git-clone. Set it to run the command 'git clone https://github.com/dgkanatsios/CKAD-exercises'</h3>

<details><summary>Answer</summary>

```bash
vi initPod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is active! && sleep 3600']
  initContainers:
  - name: init-git-clone
    image: busybox
    command: ['sh', '-c', 'git clone https://github.com/dgkanatsios/CKAD-exercises ; done;']

kubectl apply -f initPod.yaml 

```

</details>

<h2>Understand the primitives necessary to create a self-healing application</h2>

<h3> Create a pod that runs image: busybox, pass in args "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600". Set a livenessProbe to execute command 'cat /tmp/healthy'. Set an initial delay seconds to 5. Set the liveness probe to check every 5 seconds</h3>

<details><summary>Answer</summary>

```bash
vi liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

kubectl apply -f liveness.yaml
```

</details>

<h3> View the liveness-pod created above, has the liveness test failed?</h3>

<details><summary>Answer</summary>

```bash
kubectl describe pod liveness-pod
kubectl get pod liveness-pod     # has the pod restarted
```