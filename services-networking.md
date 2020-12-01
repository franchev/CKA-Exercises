# Services & Networking 20%

## Understand Host networking Configuration on the cluster nodes

## Understand connectivity between pods

## Understand clusterIP, NodePort, LoadBalancer Service types and endpoints
Kubernetes services enable communication between various components within and outside of the application. Services help us connect applications together with other applications or users.

Services Types 
- NodePort: the service makes an internal POD accessible on a port on the Node 
- Cluster IP: the service creates a virtual IP inside the cluster to enable communication between different services such as a set of front end servers to a set of back end servers 
- LoadBalancer: the service provisions a load balancer for our service in supported cloud providers

### Create a nodeport service named myapp-service pointing to nods with label type: front-end and app: myapp. Ensure that the service targetport is set to 80 & the nodeport value is set to 3000

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

### Create a deployment with the following attributes: image=nginx, replicas=3, pod label (app: myapp, type: frontend)

```bash
```

### Create another deployment with the following attributes: image=busybox, replicas=3, pod label (app: my-backend, type: mysql


```bash
```

### Create a service named redis-service of type ClusterIP that expose pod redis on port 6379

```bash
kubectl expose pod redis --port=6379 --name redis-service  -o yaml

# alternative generate yaml file
# then edit the yaml file and change the selector to point to the label of the pod
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml > svc.yaml

```

## Know how to use Ingress Controllers & Ingress Resources

## Known how to configure and use CoreDNS

## Choose an appropriate container network interface plugin

