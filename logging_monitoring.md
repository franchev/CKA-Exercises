<h1>Logging/Monitoring 5%</h1>

<h2>Understand how to monitor all cluster components</h2>

<h2>Understand how to monitor applications</h2>

<h2>Manage cluster component logs</h2>

<h2>Manage application logs</h2>

<h3>Create a pod using the image:busybox that runs this command 'i=0;while true;do echo "hello number $i"; i=$((i+1));sleep 1; done'. View its log<h3>
  
`
kubectl run busybox --image=busybox --restart=Never -- /bin/sh 'i=0;while true;do echo "hello number $i"; i=$((i+1));sleep 1; done'
kubectl logs busybox -f # similar to tail -f to follow a log
kubectl logs busybox # prints the last logs from the pod
`
