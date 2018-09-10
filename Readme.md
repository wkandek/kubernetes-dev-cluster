# Vagrant VirtualBox Kubernetes Cluster 

## Contents

####   I. Software Pre-reqs
####  II. Cluster Installation Overview 
#### III. Vagrantfile Customization
- Variable Definitions
####  IV. Cluster Installation 
- Step 1
- Step 2
- Step 3
- Step 4
####   V. Aliases
#

## I. Software Pre-reqs:
On the local machine (MacOS,Windows,Linux) install the following applications in the order listed below:

##### 1. [Vagrant](https://www.vagrantup.com/)
##### 2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
##### 3. [VirtualBox Extensions](https://download.virtualbox.org/virtualbox/5.2.18/Oracle_VM_VirtualBox_Extension_Pack-5.2.18.vbox-extpack)
##### 4. [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

## II. Cluster Installation Overview:

The Vagrantfile will install and configure a running a Single Master Node Kubernetes (latest) Cluster with a user defined number of Worker Nodes all running Ubuntu 18.04 LTS (ubuntu/bionic64).

Ubuntu provisioning scripts are embedded in the Vagrantfile. This is a fairly straight forward bash shell script. Check the echo statements in the code to understand the operations. 

User should edit variables as needed. Note there is a requirement to provide a unique token value for `KUBETOKEN`. Do not skip the Minikube pre-requisite as that is required for generating the token. 

Currently Flannel is the only network overlay the provisioning script provides. 

Kubernetes Dashboard will also be deployed with rbac token authentication. Installation instrunctions provide commands for accessing the dashboard from the local system the cluster is installed on.

## III. Vagrantfile Customization:

The following variables are defined at the top of the Vagrantfile.  Recommend using a command line text editor such as vi or nedit. `KUBEADM` must be a uniquely generated value, instructions are provided in the "Variable Definitions" section below. 

Variable Name | Default Value             |
--------------|---------------------------|
`KUBETOKEN`   | "03fe0c.e57e7831b69b2687" |
`MASTER_IP`   |     "172.16.35.100"       |
`POD_NTW_CIDR`|     "10.244.0.0/16"       |
`BOX_IMAGE`   |    "ubuntu/bionic64"      |
`NODE_COUNT`  |           3               |
`CPU`         |           1               |
`MEMORY`      |          1024             |

#### Variable Definitions

`KUBETOKEN` Generate a unique token from the Minikube VM using the following command: 

```console
$ kubectl token generate token
04ff0b.e57e683ec69b2587
```
Variable       | Definition                                                                                                  |
---------------|-------------------------------------------------------------------------------------------------------------|
`KUBETOKEN`    | Generate a unique token as described above.                                                                 |
`MASTER_IP`    | Default is "172.16.35.100"  The cluster master and apiserver IP. Do not overlap `POD_NTW_CIDR`              |
`POD_NTW_CIDR` | Default is "10.244.0.0/16"  This value is [required] for Flannel to run.                                    |
`BOX_IMAGE`    | Default is "ubuntu/bionic64" Changing OS value may require script changes.                                  |
`NODE_COUNT`   | Default is 2 Set desired number of worker nodes           |
`CPU`          | Default is 1 Recommend at least 2 if the system has the resources.                                        |
`MEMORY`       | Default is 1024 Recomend a minimum of 1024 is the system has the resources.                                |
## IV. Cluster Installation:

#### Step 1 

Download this repository 
```console
$ git clone https://github.com/ecorbett135/k8s-ubuntu-vagrant
```
#### Step 2 

Install and configure the cluster
```console
$ cd k8s-ubuntu-vagrant
$ vagrant up
```
Once installation completes  final line in output will look something like: 
   ```console
   worker3: Run 'kubectl get nodes' on the master to see this node join the cluster.
   ```
#### Step 3

ssh with port forward, check node status, start proxy and get dashboard token
(Default vagrant user password is 'vagrant'. Change it using the 'passwd' command after logging in.)
```
[LocalMachine]$ ssh -L 8001:127.0.0.1:8001 vagrant@172.16.35.100

master$ kubectl -n kube-system get nodes

NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    28m       v1.11.2
node1     Ready     <none>    27m       v1.11.2
node2     Ready     <none>    26m       v1.11.2
node3     Ready     <none>    24m       v1.11.2

$ kubectl -n kube-system describe secret $(ks get secret | awk '/^admin-user/{print $1}') | awk '$1=="token:"{print $2}'
eyJhbGciOiJSUzI1NiIsImwpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXg2OTR2Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI3Y2F1MjJjZi1iNDZkLTExZTgtOWZkMS0wMjJmNjJjZDllMjIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.PCGqgoVvJSFk8hP447cAi6VsLtvbQa_UxhdijdBK6P6i2TOfSzmTShI2gIyUGVOIiLp8RhbjbiZ_m9Cpi404dw5zKhjGcgUOUj-KpgpDgIDiO1GFeE6EHkrmni_ig0vbMF5AEemvtCdp6VS8sNqP6t-LatV-AL4S-K1i_N79wcpOCiIzdtD0itoXspz63hDt4zvRhGmLhAGIDPqT_8H79eOdxEkIjb-LmHJg6yvp0ApSCBGDJJRgDLRa-P_xS0m913EbPIK6O6gGB2zER0JB7nMdYxHByDJwKZwoZZjHp6h42f53CjKp9pjTXcufjMLyIcV80ui76PPrrB3VoWHlLQ
```
#### Step 4 

From local machine VM's are running on enter the following url or click below:

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/


## V. Aliases

Note that the master node .bashrc is configured with some aliases:
```console
alias kc='kubectl'
alias kcw='kubectl -o wide'
alias ks='kubectl -n kube-system'
alias ksw='kubectl -n kube-system -o wide'
```
