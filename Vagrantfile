#-*- mode: ruby -*-
# vi: set ft=ruby :

######################################################################################
# Variables Section. See https://github.com/ecorbett135/k8s-ubuntu-vagrant for details
######################################################################################

KUBETOKEN = "03fe0c.e57e7831b69b2687"
MASTER_IP = "172.16.35.100"
POD_NTW_CIDR = "10.244.0.0/16"
BOX_IMAGE = "ubuntu/bionic64"
NODE_COUNT = 2
CPU = 1
MEMORY = 1024

######################################################################################
# Master Node Configuration Script
######################################################################################

$masterscript = <<-SCRIPT

echo "Configuring swap space off!"
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo "Setting up ssh to allow password authetication"
mv /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
sed -e 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.orig > /etc/ssh/sshd_config
systemctl reload sshd

echo "Setting up kubernetes repository!"
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

echo "Installing: ansible docker.io kubelet kubeadm kubectl"
apt update && apt install -y ansible docker.io kubelet kubeadm kubectl
systemctl enable kubelet && systemctl restart kubelet
systemctl enable docker && systemctl restart docker

echo "Getting kube images..."
kubeadm config images pull

echo "Initializing k8s Cluster"
kubeadm init --pod-network-cidr=#{POD_NTW_CIDR} --apiserver-advertise-address=#{MASTER_IP} --token #{KUBETOKEN} --token-ttl 0

echo "Setting environment variables"
mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
echo "alias kc='kubectl'" >> /home/vagrant/.bashrc
echo "alias kcw='kubectl -o wide'" >> /home/vagrant/.bashrc
echo "alias ks='kubectl -n kube-system'" >> /home/vagrant/.bashrc
echo "alias ksw='kubectl -n kube-system -o wide'" >> /home/vagrant/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf

echo "Installing Flannel network overlay"
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
sleep 45
echo "Cleaning up Flannel daemonsets..."
kubectl -n kube-system delete ds kube-flannel-ds-arm
kubectl -n kube-system delete ds kube-flannel-ds-arm64
kubectl -n kube-system ds kube-flannel-ds-ppc64le
kubectl -n kube-system delete ds kube-flannel-ds-s390x

echo "Installing Kubernetes Dashboard"
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
sleep 45

echo "Creating Dashboard ClusterRoleBinding..."
cat << EOF > /home/vagrant/admin-user-crb.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF

kubectl create -f /home/vagrant/admin-user-crb.yaml

echo "Creating Dashboard ServiceAccount..."
cat << EOF > /home/vagrant/admin-user-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
EOF

kubectl create -f /home/vagrant/admin-user-sa.yaml
SCRIPT

######################################################################################
# Worker Node Configuration Script
######################################################################################

$workerscript = <<-SCRIPT

echo "Configuring swap space off!"
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

echo "Setting up ssh to allow password authetication"
mv /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
sed -e 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.orig > /etc/ssh/sshd_config
systemctl reload sshd

echo "Setting up kubenetes repository!"
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

echo "Installing: ansible docker.io kubelet kubeadm kubectl"
apt update && apt install -y ansible docker.io kubelet kubeadm kubectl
systemctl enable kubelet && systemctl restart kubelet
systemctl enable docker && systemctl restart docker


echo "Joining k8s cluster!"
kubeadm join --token #{KUBETOKEN} #{MASTER_IP}:6443 --discovery-token-unsafe-skip-ca-verification

SCRIPT

######################################################################################
# Create VMs Section
######################################################################################

Vagrant.configure("2") do |config|
	config.vm.define "master" do |master|
		master.vm.box = BOX_IMAGE
		master.vm.hostname = 'master'
		master.vm.network :private_network, ip: "172.16.35.100"
		master.vm.provider :virtualbox do |v|
			v.name = "master"
			v.memory = MEMORY
			v.cpus = CPU
	end
		master.vm.provision "shell", inline: $masterscript
	end

(1..NODE_COUNT).each do |i|
	config.vm.define "node#{i}" do |worker|
		worker.vm.box = BOX_IMAGE
		worker.vm.hostname = "node#{i}"
		worker.vm.network :private_network, ip: "172.16.35.10#{i}"
		worker.vm.provider :virtualbox do |v|
			v.name = "node#{i}"
			v.memory = MEMORY
			v.cpus = CPU
	end
		worker.vm.provision "shell", inline: $workerscript
  end
 end
end
