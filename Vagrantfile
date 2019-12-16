$script_general = <<SCRIPT
sudo yum update -y

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum update -y
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

sudo yum install docker-ce -y
sudo systemctl start docker
sudo systemctl enable docker
sudo groupadd docker
sudo chown root:docker /var/run/docker.sock
sudo usermod -a -G docker vagrant

cat <<EOF | sudo tee -a /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
sudo yum update -y

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

cat <<EOF | sudo tee -a /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

sudo swapoff -a
sudo sed -e '/swap/s/^/#/g' -i /etc/fstab

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl start kubelet
sudo systemctl enable kubelet
SCRIPT

$script_master = <<SCRIPT
firewall-cmd --permanent --add-port={6443,2379,2380,10250,10251,10252}/tcp
firewall-cmd --permanent --add-port={179,5473,443}/tcp
firewall-cmd --reload
SCRIPT

$script_workers = <<SCRIPT
firewall-cmd --permanent --add-port={10250,30000-32767}/tcp
firewall-cmd --permanent --add-port={179,5473,443}/tcp
firewall-cmd --reload
SCRIPT

Vagrant.configure("2") do |config|
  
  config.vm.define "master" do |master|
    master.vm.provider :virtualbox do |virtualbox|
	  virtualbox.name = "master"
	  virtualbox.memory = 2048
	  virtualbox.cpus = 2
	end
	master.vm.box = "centos/7"
	master.vm.hostname = "master"
	master.vm.network "private_network", ip: "192.168.0.10"
	master.vm.provision "shell", inline: $script_general
	master.vm.provision "shell", inline: $script_master
  end
  
  config.vm.define "worker1" do |worker1|
    worker1.vm.provider :virtualbox do |virtualbox|
	  virtualbox.name = "worker1"
	  virtualbox.memory = 1024
	  virtualbox.cpus = 1
	end
	worker1.vm.box = "centos/7"
	worker1.vm.hostname = "worker1"
	worker1.vm.network "private_network", ip: "192.168.0.20"
	worker1.vm.provision "shell", inline: $script_general
	worker1.vm.provision "shell", inline: $script_workers
  end
  
  config.vm.define "worker2" do |worker2|
    worker2.vm.provider :virtualbox do |virtualbox|
	  virtualbox.name = "worker2"
	  virtualbox.memory = 1024
	  virtualbox.cpus = 1
	end
	worker2.vm.box = "centos/7"
	worker2.vm.hostname = "worker2"
	worker2.vm.network "private_network", ip: "192.168.0.30"
	worker2.vm.provision "shell", inline: $script_general
	worker2.vm.provision "shell", inline: $script_workers
  end
end