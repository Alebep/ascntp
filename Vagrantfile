# -*- mode: ruby -*-
# vi: set ft=ruby :

# === Variables ======================================================================
Number_VMs = 0 #number VMs to launch
IP_RANGE= "192.168.56" #Change me if needed!
PUBLIC_KEY_PATH = "~/.ssh/id_rsa.pub" #Change me if needed!
READ_PUBLIC_KEY = File.read(File.expand_path(PUBLIC_KEY_PATH)).strip
PROVISION_FILES_PATH="v"

#P === rovisioning VM - New! =========================================================
PROVISION_VM = true                 #Change me if needed!
PROVISION_VM_IP = "192.168.56.10"   #Change me if needed!
PRIVATE_KEY_PATH = "~/.ssh/id_rsa"  #Change me if needed!

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = 1
  end

  (1..Number_VMs).each do |i|
    config.vm.define "server#{i}" do |node|
      node.vm.hostname = "VM#{i}"
      node.vm.network :private_network, ip: "#{IP_RANGE}.#{100+i}"
    end
  end

  if (PROVISION_VM) then
    config.vm.define "provisionVM" do |node|
      node.vm.hostname = "provisionVM"
      node.vm.network :private_network, ip: PROVISION_VM_IP 
      node.vm.provision "file", source: PRIVATE_KEY_PATH, destination: "~/.ssh/"
      node.vm.provision "file", source: "provision_files/", destination: "~/provision_files"
      node.vm.provision :shell, privileged: true, inline: $provision_x
    end
  end
end

$provision_x = <<-SHELL
  echo " === [PROVISIONER|Task 0]: Disable firewall =========================================================="
  ufw disable

  echo " === [PROVISIONER|Task 1]: Configure SSH Public Key authentication ==================================="
  echo "#{READ_PUBLIC_KEY}" >> /home/vagrant/.ssh/authorized_keys
  sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
  sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
  systemctl restart sshd

  echo " === [PROVISIONER|Task 2]: Install vim ==============================================================="
  apt-get update
  apt-get install -y vim 

  echo " === [PROVISIONER|Task 3]: Install ansible ==========================================================="
  apt-get install -y ansible

  echo " === [PROVISIONER|Task 4]: Install GCP Cli ==========================================================="
  apt-get install apt-transport-https ca-certificates gnupg
  echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
    
  curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
  apt-get update && apt-get install google-cloud-cli

  echo " === [PROVISIONER|Task 4]: Install pip ==============================================================="
  apt-get install -y pip

  echo " === [PROVISIONER|Task 5]: Install google.auth ======================================================="
  pip install requests google-auth

  echo " === [PROVISIONER|Task 6]: Install google plugins ===================================================="
  apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin

  echo " === [PROVISIONER|Task 7]: Install kubectl ==========================================================="
  apt-get install kubectl

  echo " === [PROVISIONER|Task 8]: Create cluster ============================================================"
  # ainda não está a funcionar, pois é necessário ultrapassar o problema do gcloud init necessitar de acesso
  # ao browser
  # efetuar peirmeiro o "gcloud init" manualmente, seguindo os passos lá indicados e depois executar o comando
  # seguinte também manualmente,
  # depois verificar que no GKE que o cluster foi corretamente criado.
  # ansible-playbook -i /home/vagrant/provision_files/inventory/gcp.yml/home/vagrant/provision_files/create-gke-cluster.yml

  # comando a usar mais tarde:
  # ansible-playbook -i ${PROVISION_FILES_PATH}/inventory/gcp.yml ${PROVISION_FILES_PATH}/create-gke-cluster.yml
SHELL