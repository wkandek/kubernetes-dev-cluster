#-*- mode: ruby -*-
# vi: set ft=ruby :

######################################################################################
# User Variables Section. See https://github.com/ecorbett135/k8s-ubuntu-vagrant for details
######################################################################################

KUBETOKEN = "13fe0c.e57e7831b69b2687"
VM_SUBNET = "172.16.35."
NODE_OCTET = 100
MASTER_IP = "#{VM_SUBNET}#{NODE_OCTET}"
POD_NET_CIDR = "10.244.0.0/16"
BOX_IMAGE = "ubuntu/bionic64"
WORKER_NODE_COUNT = 2
CPU = 2
MEMORY = 1024

######################################################################################
# Master Node Configuration Script
######################################################################################

$masterscript = <<-SCRIPT

echo "Turning off swap..."
swapoff -a
echo "Turning off swap at boot time..."
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


echo "Setting up ssh to allow password authetication from local machine..."
mv /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
sed -e 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.orig > /etc/ssh/sshd_config
systemctl reload sshd

echo "Setting up Kubernetes repository..."
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

echo "Setting up docker-ce repository..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

echo "Installing: ipvsadm docker kubelet kubeadm kubectl..."
apt update && apt install -y ipvsadm docker-ce=18.06.0~ce~3-0~ubuntu kubelet=1.17.3-00 kubernetes-cni=0.7.5-00  kubeadm=1.17.3-00 kubectl=1.17.3-00

echo "Enabling and Restarting services..."
systemctl enable docker && systemctl restart docker
systemctl enable kubelet && systemctl restart kubelet

echo "Getting kube images..."
kubeadm config images pull

echo "Initializing k8s Cluster..."
kubeadm init --pod-network-cidr=#{POD_NET_CIDR} --apiserver-advertise-address=#{MASTER_IP} --token #{KUBETOKEN} --token-ttl 0

echo "Setting environment variables..."
mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
echo "alias kc='kubectl'" >> /home/vagrant/.bashrc
echo "alias kcw='kubectl -o wide'" >> /home/vagrant/.bashrc
echo "alias ks='kubectl -n kube-system'" >> /home/vagrant/.bashrc
echo "alias ksw='kubectl -n kube-system -o wide'" >> /home/vagrant/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf

sleep 30

echo "Installing Flannel..."
sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

echo "Cleaning up Flannel daemonsets..."
kubectl -n kube-system delete ds kube-flannel-ds-arm
kubectl -n kube-system delete ds kube-flannel-ds-arm64
kubectl -n kube-system delete ds kube-flannel-ds-ppc64le
kubectl -n kube-system delete ds kube-flannel-ds-s390x

sleep 30

echo "Installing Kubernetes Dashboard..."
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl apply -f https://raw.githubusercontent.com/wkandek/kubernetes-dev-cluster/master/addon/dashboard/dashboard-cluster-role-binding.yaml
kubectl apply -f https://raw.githubusercontent.com/wkandek/kubernetes-dev-cluster/master/addon/dashboard/dashboard-service-account.yaml

echo "Installing Addon: Metallb (Loadbalancer)..."
kubectl apply -f https://raw.githubusercontent.com/wkandek/kubernetes-dev-cluster/master/addon/metallb/metallb-install.yaml
kubectl apply -f https://raw.githubusercontent.com/wkandek/kubernetes-dev-cluster/master/addon/metallb/layer2.config-yaml

echo "Installing Addon Web Server..."
kubectl apply -f https://raw.githubusercontent.com/wkandek/kubernetes-dev-cluster/master/addon/metallb/nginx-loadbalancer-test-deployment.yaml

echo "Starting Dashboard proxy service"
kubectl proxy &
SCRIPT

######################################################################################
# Worker Node Configuration Script
######################################################################################

$workerscript = <<-SCRIPT

echo "Turning off swap..."
swapoff -a
"Configuring swap off at boot time..."
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo "Setting up ssh to allow password authetication..."
mv /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
sed -e 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.orig > /etc/ssh/sshd_config
systemctl reload sshd

echo "Setting up kubenetes repository..."
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

echo "Setting up docker-ce repository..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

echo "Installing: docker kubelet kubeadm kubectl..."
apt update && apt install -y ipvsadm docker-ce=18.06.0~ce~3-0~ubuntu kubelet=1.17.3-00 kubernetes-cni=0.7.5-00  kubeadm=1.17.3-00 kubectl=1.17.3-00

echo "Enabling and Restarting services..."
systemctl enable docker && systemctl restart docker
systemctl enable kubelet && systemctl restart kubelet

echo "Joining k8s cluster..."
kubeadm join --token #{KUBETOKEN} #{MASTER_IP}:6443 --discovery-token-unsafe-skip-ca-verification

SCRIPT

######################################################################################
# Create VMs Section
######################################################################################

Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = BOX_IMAGE
    master.vm.hostname = 'master'
    master.vm.network :private_network, ip: "#{MASTER_IP}"
    master.vm.synced_folder "./" , "/vagrant"
    master.vm.provider :virtualbox do |v|
      v.name = "master"
      v.memory = MEMORY
      v.cpus = CPU
end
master.vm.provision "shell", inline: $masterscript
end

(1..WORKER_NODE_COUNT).each do |i|
 config.vm.define "node#{i}" do |worker|
    worker.vm.box = BOX_IMAGE
    worker.vm.hostname = "node#{i}"
    worker.vm.network :private_network, ip: "#{VM_SUBNET}#{NODE_OCTET+i}"
    worker.vm.synced_folder "./" , "/vagrant"
    worker.vm.provider :virtualbox do |v|
      v.name = "node#{i}"
      v.memory = MEMORY
      v.cpus = CPU
end
    worker.vm.provision "shell", inline: $workerscript
  end
 end
end
