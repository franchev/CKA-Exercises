<h1>Storage 7%</h1>

<h2>Understand persistent volumes and know how to create them</h2>
<h3>Create a persistent volume with path saved on the node (not recommended in production, the path must exist on the node). Set the storage capacity to 1Gi. Set the accessMode to ReadWriteOnce. </h3>
<details><summary>Answer</summary>
<p>
For this we'll need to create the persistent Volume using the yaml file. If you have issue remembering all the sections, please use the kubernetes/docs or kubectl explain persistentvolume --recursive.

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol01
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /data
```
</details>

<h2>Understand access modes for volumes</h2>
<p>
AccessModes define how a volume should be mounted on a host.
There are three possible access modes available for volumes:
ReadWriteOnce
ReadWriteMany
ReadOnlyMany
</p>

<h2>Understand persistent volume claims</h2>

<p> Once you create a persitent volume(pv), you need to set up a persistent volume claim(pvc) that binds to that volume in order to mount it on a pod. Every pvc is bound to a pv (1 to one relationship). If there are no persistent volume available to bind to, the PVC will remain in a pending state until a PV is available</p>

<h3>Create a persistent volume claim that binds to a persistent volume that has accessMode set to ReadWriteOnce. Set the PVC name to be myclaim1. Set the persistent volume claim to request 500Mi</h3>

<details><summary>Answer</summary>
<p> We will first define a yaml definition file. Remember you can use kubectl explain persistentvolumeclaim --recursive to view the options for the definition. </p>
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi

kubectl apply -f pvc.yaml # to create the pvc

kubectl get pvc or kubectl get persistentvolumeclaim          # to see the status of the pvc, if it's at pending, then there is no pv available to bind to.
```
</details>




<h2>Understand Kubernetes storage objects</h2>
<h3>Create a pod with a volume that is a directory on the node. Use image alpine and name the pod random-number. Set the pod to run command "shuf -i o-100 -n 1 >> /opt/number.out"</h3>

<details><summary>Answer</summary>
<p>
This is not recommended in a production environment, since you'll have to create the directory on the nodes prior to creating the pod. This is fine in a single node environment, but not in a multi-node.
```bash
ssh node1       # ssh to the node where the pod will be created
mkdir /data     # make the directory on the node
exit
kubectl run random-number --image=alpine --restart=Never -o yaml --dry-run -- /bin/sh "shuf -i o-100 -n 1 >> /opt/number.out" 
> random-number.yaml    # generate yaml file

edit the yaml file generated to add the volume
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: random-number
  name: random-number
spec:
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
  containers:
  - command:
    - /bin/sh -c
    - shuf -i o-100 -n 1 >> /opt/number.out
    image: alpine
    name: random-number
    volumeMounts:
    - mountPath: /opt
      name: data-volume
    resources: {}
  restartPolicy: Never

kubectl apply -f random-number.yaml       # create pod from yaml file
```

</p>
</details>

<h3>Delete the PVC created above. What happens to the PV that was created</h3>
<details><summary>Answer</summary>
```bash
kubectl delete pvc myclaim1 # deletes the pvc
kubectl get pv              # the pv sticks around until the administrator goes and deletes it, but it cannot be reused why?
```

<p>When you're creating a PV, you can set the persistenVolumeReclaimPolicy, which lets kubernetes know what to do with the PV once the PVC is deleted. By default, it is set to Retain. Here are the options
<li>Retain: keep until administrator does something with it, cannot be reused</li>
<li>Delete: automatically delete it, once the binded pvc is deleted</li>
<li>Recycle: The data in the PV will be deleted, and the PV will be made available again for other PVCs</li>
</p>
</details>

<h2>Know how to configure applications with persistent storage</h2>

<h3> Create a persistent Volume with these info:
<li>name: my-pc</li>
<li>accessMode: ReadWriteOnce</li>
<li>storage Capacity: 1Gi</li>

Create a persistent volume Claim to use the persistent volume named my-pc, set its name to be my-pvc and request storage of 500Mi. 
Create a pod name busybox that uses image busybox which mounts the pvc named my-pvc</h3>
<details><summary>Answer</summary>
<p> Let's first create the Pv by using the definition file</p>

```bash
vi my-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data

kubectl create -f my-pv.yaml
kubectl get pv   # verify that the pv is created
```

<p> next let's create the PVC to be bound to the pv my-pv</p>

```bash
vi my-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

<p> lastly, let's create a pod that will use the my-pvc claim</p>

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run > busybox.yaml    # generate the yaml file
vi busybox.yaml  # modify the yaml file to add the pvc
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  volumes:
  - name: mypvcvol
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - image: busybox
    name: busybox
    volumeMounts:
    - mountPath: "/var/www/html"
      name: mypvcvol
  restartPolicy: Never
```
</details>