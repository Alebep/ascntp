Number_Workers = 1 #number Workers to launch
IP_RANGE= "192.168.56" #Change me if needed!
PUBLIC_KEY_PATH = "~/.ssh/id_rsa.pub" #Change me if needed!
READ_PUBLIC_KEY = File.read(File.expand_path(PUBLIC_KEY_PATH)).strip

Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-20.04"
  config.vm.synced_folder ".", "/vagrant"
  config.vm.provision :shell, privileged: true, inline: $provision_all

  config.vm.define :master do |master|
    master.vm.hostname = "master"
    master.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    master.vm.network :private_network, ip: "#{IP_RANGE}.200"
    config.vm.provision :file, source: 'mysql', destination: "~/"
    config.vm.provision :file, source: 'swap', destination: "~/"
    config.vm.provision :file, source: 'persistent-volume.yml', destination: "~/"
    master.vm.provision :shell, privileged: false, inline: $provision_master_node, :args => ["#{IP_RANGE}.200"]
  end

  (1..Number_Workers).each do |i|
    config.vm.define "worker#{i}" do |worker|
      worker.vm.hostname = "worker#{i}"
      worker.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
      end
      worker.vm.network :private_network, ip: "#{IP_RANGE}.#{100+i}"
      worker.vm.provision :shell, privileged: true, inline: $provision_worker_node, :args => ["#{IP_RANGE}.#{100+i}"]
    end
  end

#  config.vm.define :master do |master|
 #   master.vm.provision :shell, privileged: false, inline: $provision_master_final
 # end

end

$provision_all = <<-SHELL
  echo "[ALL|Task 0] Disable firewall"
  ufw disable

  echo "[ALL|Task 1] Configure SSH Public Key authentication"
  echo "#{READ_PUBLIC_KEY}" >> /home/vagrant/.ssh/authorized_keys
  sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
  sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
  systemctl restart sshd

  echo "[ALL|Task 2] Update packages and install vim"
  apt-get update
  apt-get install -y vim

  echo "[ALL|Task 3] Installing Docker"
  apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt-get update
  apt-get install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
  #apt-get install docker-io -y
  echo '{"exec-opts": ["native.cgroupdriver=systemd"]}' >> /etc/docker/daemon.json

  echo "[ALL|Task 4] Enable Docker"
  systemctl enable docker

  echo "[ALL|Task 5] Restart Docker"
  systemctl restart docker

  echo "[ALL|Task 6] Download Kubernetes GPG"
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

  echo "[ALL|Task 7] Add repo of Kubernetes"
  apt-add-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
  apt-get update

  echo "[ALL|Task 8] Install Kubernetes"
  apt-get install -y kubeadm=1.18.5-00 kubelet=1.18.5-00 kubectl=1.18.5-00
  apt-mark hold kubeadm kubelet kubectl

  echo "[ALL|Task 9] Disable memory swapping"
  swapoff -a
  sed -i '/swap/d' /etc/fstab
SHELL

$provision_master_node = <<-SHELL
  OUTPUT_FILE=/vagrant/join.sh
  rm -rf $OUTPUT_FILE

  echo "[MASTER|TASK 1] Starting K8 cluster"
  sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=$1 >> $HOME/kubeinit.log 2>&1

  echo "[MASTER|TASK 2] Creating K8s directory"
  mkdir -p $HOME/.kube

  echo "[MASTER|TASK 3] Create Admin Configuration"
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  echo "[MASTER|TASK 4] Fix kubelet IP"
  echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip='$1'"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  echo "[MASTER|TASK 5] Restart kubelet"
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet

  echo "[MASTER|TASK 6] Configure flannel"
  curl -s -o kube-flannel.yml https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  sed -i.bak -e 's/^\( *\)\- \-\-kube-subnet-mgr$/&\n\1- --iface=eth1/' kube-flannel.yml
  kubectl apply -f kube-flannel.yml

  echo "[MASTER|TASK 7] Generate and save cluster join command to /vagrant/join.sh"
  kubeadm token create --print-join-command > ${OUTPUT_FILE} 2>&1
  sed -i '/WARNING/d' /vagrant/join.sh
  sudo chmod +x $OUTPUT_FILE
SHELL

$provision_worker_node = <<-SHELL
  echo "[WORKER|TASK 1] Join node to Kubernetes Cluster"
  /vagrant/join.sh

  echo "[WORKER|TASK 2] Fix kubelet IP"
  echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip='$1'"' >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  echo "[WORKER|TASK 3] Restart kubelet"
  systemctl daemon-reload
  systemctl restart kubelet

  echo "[WORKER|TASK 4] Create PV Mount Point"
  mkdir /mnt/data 
SHELL

#$provision_master_final = <<-SHELL
 # echo "[MASTER_CLUSTER_CREATION|TASK 5] Creating persistent-volume"
  #kubectl apply -f persistent-volume.yml
 
  #echo "[MASTER_CLUSTER_CREATION|TASK 6] Creating persistent-volume-clain for MySQL"
  #kubectl apply -f mysql/mysql-pvc.yml

# echo "[MASTER_CLUSTER_CREATION|TASK 7] MySQL deployment and service creation"
  # kubectl apply -f mysql/mysql-deployment.yml
  # kubectl apply -f mysql/mysql-service.yml

#  echo "[MASTER_CLUSTER_CREATION|TASK 8] SWAP deployment and service creation"
  #kubectl cordon worker1
  # kubectl apply -f swap/swap-deployment.yml
  # kubectl apply -f swap/swap-service.yml
  # kubectl uncordon worker1
# SHELL