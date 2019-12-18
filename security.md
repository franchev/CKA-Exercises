<h1>Security 12%</h1>

<h2>Know how to configure authentication and authorization</h2>



<h2>Understand Kubernetes security primitives</h2>

<h2>Know how to configure network policies</h2>


<p>by default kubernetes allow all traffics between objects. In this section, let's explore how to make this more secure</p>

<h3> Create a network policy with name: db-policy. that applies to pods with label=role=db. The policytype should be ingress from pods that matches label=name=api. The accessible port should be port 3306</h3>

<details><summary>Answer</summary>

```bash
vi networkPolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes: 
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api
    ports:
    - protocol: TCP
      port: 3306

kubectl apply -f networkPolicy.yaml

kubectl get networkpolicy to view your changes
```

</details>

<h3> </h3>

<details><summary>Answer</summary>

```bash

```

</details>


<h2>Create and manage TLS certifications for cluster components</h2>

<h2>Work with images securly</h2>

<h2>Define security contexts</h2>

<p>Capabilities are only supported at the container level and not at the POD level. If you set security contexts at the POD level it applies to all containers. If you set security contexts at both locations, the security context at the container level override the pod's</p>

<h3> Create a pod that has a container named ubuntu with image ubuntu. Set container to run command "sleep 3600" Set it to runAsUser: root with capabilities set to  MAC_ADMIN</h3>

<details><summary>Answer</summary>

```bash
vi secContextPod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 0
      capabilities:
        add: ["MAC_ADMIN"]

kubectl apply -f secContextPod.yaml
kubectl describe pod ubuntu         # check that the security context has been injested in the pod
    
```

</details>

<h2>Secure persistent key value store.</h2>