# -*- mode: ruby -*-
# vi: set ft=ruby :

#########################################
# Master Node Configuration Script
#########################################

KUBETOKEN = "02fe0c.e57e783eb69b2687"
MASTER_IP = "172.16.35.100"
POD_NTW_CIDR = "10.244.0.0/16"
BOX_IMAGE = "ubuntu/xenial64"
NODE_COUNT = 3
CPU = 4
MEMORY = 8192

$masterscript = <<-SCRIPT

swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt update && apt install -y ansible docker.io kubelet kubeadm kubectl=1.10.5-00
systemctl enable kubelet && systemctl start kubelet
systemctl enable docker && systemctl start docker

#CGROUP_DRIVER=$(sudo docker info | grep "Cgroup Driver" | awk '{print $3}')
sed -i "s|KUBELET_KUBECONFIG_ARGS=|KUBELET_KUBECONFIG_ARGS=--cgroup-driver=$CGROUP_DRIVER |g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sed -i 's/10.96.0.10/10.3.3.10/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload
systemctl stop kubelet && systemctl start kubelet

kubeadm config images pull
kubeadm init --pod-network-cidr=#{POD_NTW_CIDR} --apiserver-advertise-address=#{MASTER_IP} --token #{KUBETOKEN} --token-ttl 0

mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
sleep 90
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

SCRIPT

#########################################
# Worker Node Configuration Script
#########################################

$workerscript = <<-SCRIPT

swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt update && apt install -y ansible docker.io kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
systemctl enable docker && systemctl start docker

CGROUP_DRIVER=$(sudo docker info | grep "Cgroup Driver" | awk '{print $3}')
sed -i "s|KUBELET_KUBECONFIG_ARGS=|KUBELET_KUBECONFIG_ARGS=--cgroup-driver=$CGROUP_DRIVER |g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sed -i 's/10.96.0.10/10.3.3.10/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload
systemctl stop kubelet && systemctl start kubelet
kubeadm join --token #{KUBETOKEN} #{MASTER_IP}:6443 --discovery-token-unsafe-skip-ca-verification

SCRIPT

#########################################
# Create VMs Section
#########################################

Vagrant.configure("2") do |config|
	config.vm.define "master" do |master|
		master.vm.box = BOX_IMAGE
		master.vm.hostname = 'master'
		master.vm.network :private_network, ip: "172.16.35.100"
		master.vm.network :public_network, :bridge => "en1: Wi-Fi (Wireless)"
		master.vm.provider :virtualbox do |v|
			v.name = "master"
			v.memory = MEMORY
			v.cpus = CPU
	end
		master.vm.provision "shell", inline: $masterscript
	end

(1..NODE_COUNT).each do |i|
	config.vm.define "worker#{i}" do |worker|
		worker.vm.box = BOX_IMAGE
		worker.vm.hostname = "node#{i}"
		worker.vm.network :private_network, ip: "172.16.35.10#{i}"
		worker.vm.network :public_network, :bridge => "en1: Wi-Fi (Wireless)"
		worker.vm.provider :virtualbox do |v|
			v.name = "node#{i}"
			v.memory = MEMORY
			v.cpus = CPU
	end
		worker.vm.provision "shell", inline: $workerscript
  end
 end
end

