# Vagrant VirtualBox Kubernetes Cluster 

## Prerequisites:

1. Install [Vagrant](https://www.vagrantup.com/)
2. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
3. Install [VirtualBox Extensions](https://download.virtualbox.org/virtualbox/5.2.18/Oracle_VM_VirtualBox_Extension_Pack-5.2.18.vbox-extpack)
4. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

## Overview:

The Vagrantfile will install and configure a running Kubernetes (latest) cluster on Ubuntu 16.04

Ubuntu provisioning scripts embedded in Vagrant file. User must provide a KUBETOKEN.

Flannel network overlay will be installed automatically. 

Kubernetes Dashboard will also be installed 

## Customize the Vagrantfile before Installation:

The following variables are defined at the top of the Vagrantgfile.  Recommend using a command line text editor such as vi or nedit. KUBEADM must be a uniquely generated value, instructions are provided in the "Variable Definitions" section below. 

Variable Name | Default Value             |
--------------|---------------------------|
KUBETOKEN     | "02fe0c.e57e783eb69b2687" |
MASTER_IP     |     "172.16.35.100"       |
POD_NTW_CIDR  |     "10.244.0.0/16"       |
BOX_IMAGE     |    "ubuntu/xenial64"      |
NODE_COUNT    |           3               |
CPU           |           1               |
MEMORY        |          512              |

## Variable Definitions

`KUBETOKEN` Generate a unique token from the Minikube VM using the following command: 

```console
$ kubectl token generate token
04ff0b.e57e683ec69b2587
```
Variable | Definition|
----------------------
`CPU` Default is 1.  Recommend at least 2 if the system has the resources.

`MEMORY` Default is 512. Recomend a minimum of 1024 is the system has the resources. 

`NODE_COUNT` Default is 2. Set the desired number of worker nodes (Note: There is no variable for master node):

`MASTER_IP` Default is "172.16.35.100" . This will be the cluster master and apiserver IP. Do not overlap `POD_NTW_CIDR`.

`POD_NTW_CIDR` Default is "10.244.0.0/16". This value is [required] for Flannel to run.
 
`BOX_IMAGE` Default is "ubuntu/xenial64" . Changing OS value is not recommended as scripts may break. 

## Cluster Installation:
```console
$ git clone https://github.com/ecorbett135/k8s-ubuntu-vagrant

$ cd k8s-ubuntu-vagrant

$ vagrant up
# Once installation completes  final line in output will look something like: 
    worker3: Run 'kubectl get nodes' on the master to see this node join the cluster.

$ vagrant ssh master

$ kubectl -n kube-system get nodes

NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    28m       v1.11.2
node1     Ready     <none>    27m       v1.11.2
node2     Ready     <none>    26m       v1.11.2
node3     Ready     <none>    24m       v1.11.2
```





