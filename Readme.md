# Vagrant VirtualBox Ubuntu Kubernetes Cluster with Dashboard

### Table of Contents

   ####   I. Software Pre-reqs
   ####  II. Cluster Installation Overview
   - Table 1. List of nodes and IP Addresses"
   #### III. Vagrantfile Customization
   - Variable Definitions
   ####  IV. Cluster Installation 
   - Step 1  Download this repository
   - Step 2  Edit Variables, install and configure cluster 
   - Step 3  Login and validate cluster is running
   - Step 4  Access Dashboard
   ####   V. Aliases
   #

## I. Software Pre-reqs:
On the local machine (Mac OS, Windows, Linux) install the following applications in the order listed below:

##### 1. [Vagrant](https://www.vagrantup.com/)
##### 2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
##### 3. [VirtualBox Extensions](https://download.virtualbox.org/virtualbox/5.2.18/Oracle_VM_VirtualBox_Extension_Pack-5.2.18.vbox-extpack)
##### 4. [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
##### 5. [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)


## II. Cluster Installation Overview:

The Vagrantfile will install and configure a running Kubernetes (latest) Cluster. The cluster will be comprised of a Single Master Node with a user defined number of Worker Nodes. All nodes will run the Linux distribution Ubuntu 18.04 LTS (ubuntu/bionic64) in a Virtualbox Virtual Machine.

There is a private internal network `172.16.35.0/24` created and all nodes are assigned a static address. The nodes can be accessed using the upcoming command example when run from the same directory the `vagrant up` command was executed from during installation. (Replace `NodeName` from "Table 1. List of nodes and IP Addresses" table):

```
[LocalMachine]$ vagrant ssh NodeName
```
- Table 1. List of nodes and IP addresses

NodeName  |  IPAddr       |
----------|---------------|
master    | 172.16.35.100 |
node1     | 172.16.35.101 |
node2     | 172.16.35.102 |
node3     | 172.16.35.103 |

If more than three worker nodes were created the pattern would continue `node4` with ip `172.16.35.104` and so forth. Note that the nodes /etc/ssh/sshd_config file has been modified to allow ssh login for ip 172.16.35.10x. This will be shown with the master `IPAddr` in a later example and is possible with any node's `IPAddr`.

Cluster provisioning scripts for the master and worker nodes are embedded in the Vagrantfile. These are fairly straight forward bash shell scripts: `$masterscript` and `$workerscript`. Check the echo statements in the code to understand the operations. 

User should edit variables as needed. Note there is a requirement to provide a unique token value for `KUBETOKEN`. Do not skip the Minikube pre-requisite as that is required for generating the token. 

Currently Flannel is the only network overlay the provisioning script provides. 

Kubernetes Dashboard will also be deployed with rbac token authentication. Installation instructions provide commands for accessing the dashboard from the local system the cluster is installed on.

## III. Vagrantfile Customization:

"Table 2. Variable Defaults" displays the default values for the variables defined in the Vagrantfile. These should be edited as prescribed in "Table 3. Variable Definitions".For Linux, Mac OS, use a command line text editor like vi. For Windows, Notepad++. 

IMPORTANT: `KUBETOKEN` must be a uniquely generated value, instructions are provided in the "Variable Definitions" section below. 

- Table 2. Variable Defaults

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
[Minikube]$ kubectl token generate token
04ff0b.e57e683ec69b2587
```
- Table 2 Variable Definitions

Variable       | Definition                                                                                                  |
---------------|-------------------------------------------------------------------------------------------------------------|
`KUBETOKEN`    | Generate a unique token as described above, copy and paste value replacing default value in Vagrantfile.    |
`MASTER_IP`    | Default is `"172.16.35.100"`. Cluster Master and Apiserver IPAddr. Do not overlap `POD_NTW_CIDR`            |
`POD_NTW_CIDR` | Default is `"10.244.0.0/16"`. This value is [required] for Flannel to run.                                  |
`BOX_IMAGE`    | Default is `"ubuntu/bionic64"`. Changing OS value may require script changes.                               |
`NODE_COUNT`   | Default is `2` Set desired number of worker nodes                                                           |
`CPU`          | Default is `1`. Recommend at least `2` if the system has the resources.                                     |
`MEMORY`       | Default is `1024`. Recomend a minimum of `1024`. More per VM is better if the system has the resources.     |
## IV. Cluster Installation:

#### Step 1 

Open a terminal session on Download this repository 

```
$ git clone https://github.com/ecorbett135/k8s-ubuntu-vagrant
```

#### Step 2 
Ensure all variables have been edited to desired values and `KUBETOKEN` is a uniquely generated token value.
Install and configure the cluster:

```
$ cd k8s-ubuntu-vagrant
$ vagrant up
```
Once installation completes  final line in output will look something like: 
   ```console
   worker3: Run 'kubectl get nodes' on the master to see this node join the cluster.
   ```
#### Step 3
Login using ssh with port forward, check node status, start proxy and get dashboard token (copy to paste into web browser)
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

Now select `TOKEN` and past token copied from terminal on the provided line.


## V. Aliases

Note that the master node .bashrc is configured with some aliases:
```console
alias kc='kubectl'
alias kcw='kubectl -o wide'
alias ks='kubectl -n kube-system'
alias ksw='kubectl -n kube-system -o wide'
```
