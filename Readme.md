Vagrant VirtualBox Kubernetes Cluster 

Prerequisites:

1. Install Vagrant
2. Install VirtualBox
  a. Install VirtualBox Extensions

Overview:

The Vagrantfile will install and configure a running Kubernetes (latest) cluster on Ubuntu 16.04

Ubuntu provisioning scripts embedded in Vagrant file. User must provide a KUBETOKEN.

Flannel network overlay will be installed automatically.  

Cluster Installation:

git clone https://github.com/ecorbett135/k8s-ubuntu-vagrant

cd k8s-ubuntu-vagrant

vagrant up

vagrant ssh master

kubectl -n kube-system get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    28m       v1.11.2
node1     Ready     <none>    27m       v1.11.2
node2     Ready     <none>    26m       v1.11.2
node3     Ready     <none>    24m       v1.11.2

