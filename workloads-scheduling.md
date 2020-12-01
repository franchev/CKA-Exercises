# Workloads & Scheduling 15%

## Understand deployments and how to perform rolling update and rollbacks
There are 2 types of deployment strategy

- Recreate Strategy: not the default strategy, because this involves the deletion of all pods prior to recreating them. This means that your app is down as it's beeing updated</li>
- Rolling Update Strategy: This is the default strategy. This involves taking one pod down at a time, so that the application is working while being upgraded</li>

### Create a deployment named nginx running image: nginx:1.7.1

```bash
kubectl run nginx image=nginx:1.7.1
kubectl get deploy   # to see the list of deployments and their status
```

### Change the image of the deployment created above to be nginx:1.8.1

```bash
kubectl set image deployment/nginx nginx=nginx:1.8.1
```

### Rollback to the previous deployment version with image: nginx:1.7.1

```bash
kubectl rollout undo deployment/nginx
kubectl rollout status deployment/myapp     # allows you to view the status of the update
kubectl rollout history deployment/myapp    # gives you a list of past updates
```

### Rollback to the 2nd version of the deployment  with image: nginx:1.8.1

```bash
kubectl rollout undo deployment/nginx --revision=2
```

## use configMaps and Secrets to configure applications

## Know how to scale applications

## Understand the primitives used to create robust, self-healing application deployments

## Understand how resource limits can affect Pod Scheduling

The default value for CPU=0.5 and Mem=256Mi. If you don't specify, kubernetes will set your containers default resources requests to these values. You can go as low as 1m for CPU.

### Create a pod named webapp-color, image=nginx with a resource requests memory value of 1Gi and CPU=1. Also set the limits to cpu=2 & memory= 2Gi

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

## Awareness of manifest management and common templation tools
