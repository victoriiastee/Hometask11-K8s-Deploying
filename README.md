# Hometask11-K8s-Deploying

### Execution:
1. Prepare 2 VMs (kubemaster and kubenode) with the same parametrs (4 CPU, 8 GB RAM, Ubuntu)
![image](https://user-images.githubusercontent.com/102364456/216151063-fb29a747-fcc8-45e2-8ea8-9986a144da9d.jpg)
<img src="https://user-images.githubusercontent.com/102364456/216150879-b280000e-1a1b-4a52-aeae-12a50ba98ceb.jpg" height="600"/></h1>
<img src="https://user-images.githubusercontent.com/102364456/216150769-58a82ead-2585-453d-9572-334945fd8d73.jpg" height="400"/></h1>

2. Connect via SSH to VMs
![image](https://user-images.githubusercontent.com/102364456/216151135-3194516d-390c-47de-b0cd-b6738209537f.jpg)
#### 3.Run commands in two VMs (kubemaster and kubenode)
```
sudo apt update
sudo apt upgrade -y
```
> __Note!__ 
***apt-get update*** downloads the package lists from the repositories and "updates" them to get
information on the newest versions of packages and their dependencies. 
***apt-get upgrade***  will fetch new versions of packages existing on the machine if APT knows
about these new versions by way of apt-get update. 


Edit the hosts file with the command: `sudo nano /etc/hosts` 

![image](https://user-images.githubusercontent.com/102364456/216151206-67b57653-144b-4967-98eb-f75b42a687bc.jpg) 

Install the first dependencies with: `sudo apt install curl apt-transport-https -y` 

![image](https://user-images.githubusercontent.com/102364456/216151264-91f87cf6-01e6-4ae5-97c8-66debd441311.jpg) 

Next, add the necessary GPG key with the command: 
`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -` 

Add the Kubernetes repository with:  


`echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list` 


Update apt: `sudo apt update`  


Install the required software with the command: `sudo apt -y install vim git curl wget kubelet kubeadm kubectl`  


Place kubelet, kubeadm, and kubectl on hold with: `sudo apt-mark hold kubelet kubeadm kubectl` 


![image](https://user-images.githubusercontent.com/102364456/216155018-0725c3e9-6070-462f-8d1b-5090ab6857f8.png) 

Start and enable the kubelet service with: `sudo systemctl enable --now kubelet` 
Next, we need to disable swap on both kubemaster. Open the fstab file for editing with:
```
sudo nano /etc/fstab
```
Save and close the file.  


![image](https://user-images.githubusercontent.com/102364456/216151459-674b391a-7cc4-48c6-91aa-a129799e4654.jpg) 

You can either reboot to disable swap or simply issue the following
command to finish up the job: `sudo swapoff -a` 
Enable Kernel Modules and Change Settings in sysctl:
```
sudo modprobe overlay
sudo modprobe br_netfilter
``` 

Change the sysctl settings by opening the necessary file with the command: `sudo nano /etc/sysctl.d/kubernetes.conf`
Look for the following lines and make sure they are set as you see below:
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```
![image](https://user-images.githubusercontent.com/102364456/216151594-c7391cad-b6a2-47eb-a2da-6e3f7d5b2560.jpg) 

Save and close the file. Reload sysctl with: `sudo sysctl --system`
### Install containerd
Install the necessary dependencies with:
`sudo apt install curl gnupg2 software-properties-common apt-transport-https ca-certificates -y` 

![image](https://user-images.githubusercontent.com/102364456/216151752-e23ea772-0ab0-410f-b98f-9b7547ac9dc4.jpg) 

Add the GPG key with: `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -` 

![image](https://user-images.githubusercontent.com/102364456/216151903-38481a1a-a4c7-4d18-bf18-1bb23932454a.jpg) 

Add the required repository with: 
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```  

Install containerd with the commands:
```
sudo apt update
sudo apt install containerd.io -y
```
![image](https://user-images.githubusercontent.com/102364456/216152189-1ac9c32e-320b-4c0a-bccd-82bd626b080c.jpg)

Change to the root user with: `sudo su` 

Create a new directory for containerd with: `mkdir -p /etc/containerd` 

Generate the configuration file with: `containerd config default>/etc/containerd/config.toml` 

Exit from the root user with:`exit` 

Restart containerd with the command:`sudo systemctl restart containerd` 

Enable containerd to run at startup with: `sudo systemctl enable containerd` 

### Initialize the Master Node 

Pull down the necessary container images with: `sudo kubeadm config images pull` 

![image](https://user-images.githubusercontent.com/102364456/216152335-9f2e3ced-217c-4f18-87d0-206e952d767f.jpg)
### Command only for kubemaster :
Now, using the kubemaster IP address initialize the master node with: `sudo kubeadm init --pod-network-cidr=192.168.0.0/16`
Finally, you need to create a new directory to house a configuration file and give it the proper permissions which is done with the following commands:
```
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
![image](https://user-images.githubusercontent.com/102364456/216152603-b7da7066-fd2a-45e5-9a9a-b3d730bd1fd1.jpg) \ 

List Kubernetes Nodes: `kubectl get nodes`
### Command only for kubenode :
Connect kubenode to kubemaster: `sudo su` 
```
kubeadm join 10.128.0.4:6443
--token eysnwi.xdsee2pi8cacg9en
-discovery-token-ca-cert-hashsha256:14535d463c8800a66a441161c2bbb84914389b40a4a828a30ac329c7a6a9b
``` 
(copy from kubemaster output)

![image](https://user-images.githubusercontent.com/102364456/216152730-35446d67-afa2-4e11-818f-3ef1457af0da.jpg) 

### Comeback to kubemaster :
Install network: 
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml -O
kubectl create -f custom-resources.yaml
``` 

Wait when all pods will be ready: `kubectl get pods --all-namespaces -w` 

![image](https://user-images.githubusercontent.com/102364456/216152920-1e8e1346-8182-43ac-ab78-f6fda87a1da2.jpg) 


### Result: 

`kubectl get nodes -o wide` 

![image](https://user-images.githubusercontent.com/102364456/216153068-873eddfa-7ab8-49d4-a6d4-caa9e7abf5ff.jpg) 

Successfully deployed Kubernetes!

 

