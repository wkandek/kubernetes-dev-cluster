# Vagrant VirtualBox Kubernetes Cluster 

## Prerequisites:

1. Install [Vagrant](https://www.vagrantup.com/)
2. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
3. Install [VirtualBox Extensions](https://download.virtualbox.org/virtualbox/5.2.18/Oracle_VM_VirtualBox_Extension_Pack-5.2.18.vbox-extpack)
4. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/

## Overview:

The Vagrantfile will install and configure a running Kubernetes (latest) cluster on Ubuntu 16.04

Ubuntu provisioning scripts embedded in Vagrant file. User must provide a KUBETOKEN.

Flannel network overlay will be installed automatically. 

Kubernetes Dashboard will also be installed 

## Customization:

Variables are defined at the top of the Vagrantgfile. Recommend using a command line text editor such as vi or nedit. 

[The variables that require user customization:]

{KUBETOKEN}
Generate from another kubernetes cluster by running the following command in Minikube VM':
$ kubectl token generate token

CPU
Default is 4.  Depending on the number of Physical cores on 


MEMORY = 8192
 

This will be the cluster master and apiserver IP: 
MASTER_IP = "172.16.35.100"

THe following is required for Flannel to run:
POD_NTW_CIDR = "10.244.0.0/16"

Operating System image - suggest leaving the default:
BOX_IMAGE = "ubuntu/xenial64"

Set the number of worker nodes (Note: There is no variable for master node):
NODE_COUNT = 3

Set VM compute resources for all cluster nodes:



## Cluster Installation:

$ git clone https://github.com/ecorbett135/k8s-ubuntu-vagrant

$ cd k8s-ubuntu-vagrant

$ vagrant up

...
    worker3: Run 'kubectl get nodes' on the master to see this node join the cluster.

real	7m4.092s .  <<< This is an actual build time of a 4 node cluster
user	0m12.755s
sys	0m7.203s


$ vagrant ssh master

$ kubectl -n kube-system get nodes

NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    28m       v1.11.2
node1     Ready     <none>    27m       v1.11.2
node2     Ready     <none>    26m       v1.11.2
node3     Ready     <none>    24m       v1.11.2






