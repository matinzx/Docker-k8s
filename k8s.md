# Setting Up Kubernetes on Three Virtual Machines (VMs) using Kubeadm

This guide walks you through setting up a Kubernetes cluster with three virtual machines (VMs), consisting of one master node and two worker nodes. We'll also go through some basic examples of deploying applications on the cluster.

---

## 1. Prerequisites

Before starting, ensure that:

- **All VMs are properly configured**: Each of the three VMs should be running Ubuntu 24 and should be able to communicate with each other.
- **Root access**: You’ll need root or `sudo` access on all VMs.
- **VM names and IPs**: Ensure that each VM has a unique name and static IP address, and they can communicate with each other over the network.

---

## 2. Installing Prerequisites on All VMs

Let's install the necessary tools on each VM.

### Step 1: Update the System

To ensure that your system is up to date, run:

```bash
sudo apt update && sudo apt upgrade -y
Step 2: Install Docker
Docker is required for running containers in Kubernetes. Install Docker by running:

bash
Copy
sudo apt install -y docker.io
After installation, enable and start Docker:

bash
Copy
sudo systemctl enable docker
sudo systemctl start docker
Step 3: Install Kubeadm, Kubelet, and Kubectl
To install the Kubernetes tools (kubeadm, kubelet, and kubectl), run the following commands:

bash
Copy
sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubeadm kubelet kubectl
Step 4: Disable SWAP
Kubernetes requires SWAP to be disabled. You can disable SWAP temporarily with:

bash
Copy
sudo swapoff -a
To disable SWAP permanently, edit the /etc/fstab file and remove the SWAP entry.

3. Setting Up the Master Node
Now, let's set up the master node on one of the VMs.

Step 1: Initialize the Cluster with Kubeadm
Run the following command on the master node to initialize the Kubernetes cluster:

bash
Copy
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
This command initializes the Kubernetes cluster with a pod network CIDR of 10.244.0.0/16 (this is necessary for networking between pods). After the process is complete, you'll receive output with instructions for joining the worker nodes to the cluster.

Step 2: Configure Kubectl
To configure kubectl to manage the cluster, run the following commands on the master node:

bash
Copy
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Step 3: Install Networking for Pods
Kubernetes pods need a network plugin to communicate with each other. Flannel is a widely used network plugin for this purpose. Install Flannel by running the following command:

bash
Copy
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
4. Joining the Worker Nodes
After setting up the master node, you need to join the worker nodes to the cluster.

Step 1: Run the Join Command
On the master node, you’ll receive a command like the following to join the worker nodes:

bash
Copy
kubeadm join <MASTER_IP>:<PORT> --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
Run this command on each of the worker nodes. This will connect them to the master node and allow them to join the cluster.

Step 2: Verify the Worker Nodes Have Joined
After running the kubeadm join command on the worker nodes, verify that they have joined the cluster by running the following command on the master node:

bash
Copy
kubectl get nodes
You should see output like this:

pgsql
Copy
NAME             STATUS   ROLES    AGE     VERSION
master-node      Ready    master   10m     v1.23.4
worker-node-1    Ready    <none>   5m      v1.23.4
worker-node-2    Ready    <none>   5m      v1.23.4
This shows that all three nodes (master and two worker nodes) are part of the cluster.

5. Managing the Cluster and Deploying Applications
Now that the cluster is set up, you can deploy applications and manage your resources.

Example 1: Deploying an Nginx Application
You can deploy a simple Nginx application with 3 replicas by running:

bash
Copy
kubectl run myapp --image=nginx --replicas=3
This command creates a deployment with 3 replicas of Nginx pods. To check the status of the pods, run:

bash
Copy
kubectl get pods
You should see something like this:

sql
Copy
NAME                      READY   STATUS    RESTARTS   AGE
myapp-5b6d68d6d9-4k6g9     1/1     Running   0          2m
myapp-5b6d68d6d9-hk2g7     1/1     Running   0          2m
myapp-5b6d68d6d9-p9p7z     1/1     Running   0          2m
Example 2: Exposing the Nginx Application
To expose the Nginx application to the outside world, you can create a service of type NodePort:

bash
Copy
kubectl expose deployment myapp --type=NodePort --port=80
This command will expose the Nginx app on a random port between 30000 and 32767. To check which port the service is exposed on, run:

bash
Copy
kubectl get svc
You should see something like this:

pgsql
Copy
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
myapp        NodePort    10.96.152.137   <none>        80:31387/TCP   3m
You can access the Nginx app by going to http://<VM_IP>:31387 in your browser.

Conclusion
You now have a fully functional Kubernetes cluster with three VMs: one master node and two worker nodes. You can start using this cluster to deploy and manage containerized applications.

