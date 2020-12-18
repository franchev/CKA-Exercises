# Cluster Architecture, Installation & Configuration 25%

## Manage Role Based Access Control (RBAC)

## Use kubeadm to install a basic cluster

Since we are allowed to have one extra chrome tab with the kubernetes documentation during the exam, we are going to use that to setup our basic cluster. 

### For this to work, we'll need these requirements:
1. VirtualBox
2. Vagrant
3. git

I'm using a mac, so my instructions will be for Mac. You can find instructions for installing the requirements for your specific OS.
- To install Virtualbox, we'll go here: https://www.virtualbox.org/wiki/Downloads
- Under VirtualBox X.X.X platform packages, click on the package that applies to you. In my case it's OS X hosts
- This download a .dmg file, I'll double click & follow the installation steps on the mac.

- To install vagrant, I will be using homebrew from the terminal
- If you don't have homebrew install, please go here: https://brew.sh/ to copy the install oneliner. In my case, it looks like this:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- Now let's install vagrant
```bash
brew install vagrant
```

- Let's install git (if you don't have it already)
```bash
brew install git
```

### Now that our requirements are installed, Let's download the Vagrantfile that we'll use to setup our cluster
I was following the steps from this Udemy class: https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests. 
The instructor provided an already build Vagrantfile that we can use for a cluster using 1 master node & 2 worker nodes on Ubuntu. 
Let's then use that VagrantFile
- Clone the repository (if you don't have a github account, you can download the zip file instead.)
```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course
```

- Let's check the status of our vagrant boxes
```bash
vagrant status
Current machine states:

kubemaster                not created (virtualbox)
kubenode01                not created (virtualbox)
kubenode02                not created (virtualbox)
```

- Let's bring up the vagrant boxes (this might take a few minutes, mine took 5 minutes)
```bash
vagrant up
```

### Now we can follow the instructions from the kubernetes.io docs.

- Please go here: https://kubernetes.io/docs/home/
- search for kubeadm install
- Click on the first link: Installing kubeadm
- There are some steps that we will perform on all nodes in the cluster & there are a few steps that are specific to the master node. 

- Verify the MAC address & prodct_UUID are unique on every node (please run this step on all nodes, if you can have multiple terminal open at the same time, this would be useful. However during the exam, you're not given multiple terminals, so remember to ssh to the different nodes.)
```bash
vagrant ssh kubemaster
ifconfig -a
sudo cat /sys/class/dmi/id/product_uuid
```

```bash 
vagrant ssh kubenode01
ifconfig -a
sudo cat /sys/class/dmi/id/product_uuid
```

```bash 
vagrant ssh kubenode02
ifconfig -a
sudo cat /sys/class/dmi/id/product_uuid
```

- Let iptables see bridged traffic (perform on all nodes)
```bash
lsmod | grep br_netfilter     # check if br_netfilter module is loaded, if nothing is outputed, then it's not loaded
sudo modprobe br_netfilter

# ensure net.bridge.bridge-nf-call-iptables is set to 1
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```


- Now we need to install a container runtime, we'll use docker for this. Please click on the link Container runtimes: https://kubernetes.io/docs/setup/production-environment/container-runtimes/. Let's scroll to the docker section and follow the steps to install docker containers. (Perform on all nodes)

```bash
# (Install Docker CE)
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
sudo apt-get update && sudo apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2

# Add Docker's official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -

# Add the Docker apt repository:
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# Install Docker CE
sudo apt-get update && sudo apt-get install -y \
  containerd.io=1.2.13-2 \
  docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)

# Set up the Docker daemon
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Create /etc/systemd/system/docker.service.d
sudo mkdir -p /etc/systemd/system/docker.service.d

# Restart Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

sudo systemctl enable docker
```

- Let's return back to the install-kubeadm page, we will now install kubeadm, kubelet & kubectl (perform on all nodes)
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

- Scroll to the bottom of the page, in What's next, let's click the Using kubeadm to create a Cluster link
- Let's Initialize the control pane on the master node. We will use this cidr block for our pod network: 10.10.244.0/16 (this might take a few minutes, perform on master node only)

```bash
# let's get the master node IP
sudo ifconfig enp0s8 | grep -I net | grep -v inet6 | awk '{print $2}'

# let's run the initialize command (please replace the api-server-advertize-address with the ip you got from above)
sudo kubeadm init --pod-network-cidr 10.244.0.0/16 
```

- Now in the output, please copy the command starting with kubeadm join... in a text editor, we will use that on the worker nodes to get them to join later

- Run these command to start using the cluster, You can copy these from your outputs from the initialize command
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

- We now need a pod network, we will use the weave pod network, I got this information by going here: https://kubernetes.io/docs/concepts/cluster-administration/addons/. Then clicking on the Weave Net link. 
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl get nodes # this is to verify that the master node is ready in the cluster. You will not see the other nodes until you've joined them
```

- Let's now join the 2 other nodes to the cluster (please run this on kubenode01 & kubenode02 )
```bash
# please replace this command with the command that you copied earlier in your text editor. 
sudo kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

kubectl get nodes  # You should now see all the nodes, they might not be ready, but if you run this command a bunch of times, they'll be ready (mine took about 5 minutes for them to be ready)
```

- Our cluster is now ready, let's create a basic nginx pod (perform on any node)
```bash
kubectl run nginx --image=nginx

kubectl get pods       # you should now see your pod

# let's delete it
kubectl delete pod nginx
```

## Provision underlying infrastructure to deploy a kubernetes cluster

## Perform a version upgrade on a kubernetes cluster using kubeadm

## Implement etcd backup and restore
