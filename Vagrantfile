# -*- mode: ruby -*-
# vi: set ft=ruby :

# === Variables ======================================================================
IP_RANGE= "192.168.56" #Change me if needed!
PUBLIC_KEY_PATH = "~/.ssh/id_rsa.pub" #Change me if needed!
READ_PUBLIC_KEY = File.read(File.expand_path(PUBLIC_KEY_PATH)).strip
PROVISION_FILES_PATH="/home/vagrant/provision_files"

#P === rovisioning VM - New! =========================================================
PROVISION_VM = true                 #Change me if needed!
PROVISION_VM_IP = "192.168.56.10"   #Change me if needed!
PRIVATE_KEY_PATH = "~/.ssh/id_rsa"  #Change me if needed!

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  if (PROVISION_VM) then
    config.vm.define "provisionVM" do |node|
      node.vm.hostname = "provisionVM"
      node.vm.network :private_network, ip: PROVISION_VM_IP 
      node.vm.provision "file", source: PRIVATE_KEY_PATH, destination: "~/.ssh/"
      node.vm.provision "file", source: "private_provision_files/", destination: "~/provision_files"
      node.vm.provision "file", source: "public_provision_files/", destination: "~/provision_files"
      node.vm.provision "file", source: "google_ssh_keys/", destination: "~/.ssh"
      node.vm.provision :shell, privileged: true, inline: $provision_x
      node.vm.provision :shell, privileged: false, inline: $provision_y
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

  echo " === [PROVISIONER|Task 4]: Install pip (python package managar) ======================================="
  apt-get install -y pip

  echo " === [PROVISIONER|Task 5]: Install google.auth ======================================================="
  pip install requests google-auth

  echo " === [PROVISIONER|Task 6]: Install ansible-core ======================================================"
  pip install ansible-core

  echo " === [PROVISIONER|Task 7]: Install google plugins ===================================================="
  apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin

  echo " === [PROVISIONER|Task 8]: Install kubectl ==========================================================="
  apt-get install kubectl

  echo " === [PROVISIONER|Task 9]: Install sqlite3 ==========================================================="
  apt-get install sqlite3

  echo " === [PROVISIONER|Task 10]: Activate the use of kubectl ==============================================="
  echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> /home/vagrant/.bashrc
  echo "export PATH=$PATH:/home/vagrant/.local/bin" >> /home/vagrant/.bashrc
  
  source /home/vagrant/.bashrc
  apt-get update && apt-get --only-upgrade -y install google-cloud-sdk-bigtable-emulator google-cloud-sdk-datalab google-cloud-sdk-terraform-tools google-cloud-sdk-nomos google-cloud-sdk-gke-gcloud-auth-plugin google-cloud-sdk-spanner-emulator google-cloud-sdk-harbourbridge google-cloud-sdk-app-engine-python google-cloud-sdk-cloud-run-proxy google-cloud-sdk-kpt google-cloud-sdk kubectl google-cloud-sdk-config-connector google-cloud-sdk-local-extract google-cloud-sdk-kubectl-oidc google-cloud-sdk-package-go-module google-cloud-sdk-app-engine-python-extras google-cloud-sdk-app-engine-grpc google-cloud-sdk-cloud-build-local google-cloud-sdk-log-streaming google-cloud-sdk-firestore-emulator google-cloud-sdk-cbt google-cloud-sdk-skaffold google-cloud-sdk-app-engine-go google-cloud-sdk-anthos-auth google-cloud-sdk-minikube google-cloud-sdk-datastore-emulator google-cloud-sdk-app-engine-java google-cloud-sdk-pubsub-emulator
 SHELL

$provision_y = <<-SHELL
  echo " === [PROVISIONER|Task 11]: Install ansible modules =================================================="
  ansible-galaxy collection install kubernetes.core
  ansible-galaxy collection install google.cloud

  echo " === [PROVISIONER|Task 12]: Install openshift ========================================================"
  pip install openshift

  #Não fazer este
  #pip install pyyaml kubernetes 

  echo " === [PROVISIONER|Task 13]: GCloud Init  ============================================================="
  # gcloud init
  # Fazer manualmente depois do gcloud init
  # gcloud container clusters get-credentials ascn-cluster

  echo " === [PROVISIONER|Task 14]: Create cluster =========================================================="
  echo " !!! ----------------- Option not available, run it manually"
  # ainda não está a funcionar, pois é necessário ultrapassar o problema do gcloud init necessitar de acesso
  # ao browser
  # efetuar peirmeiro o "gcloud init" manualmente, seguindo os passos lá indicados e depois executar o comando
  # seguinte também manualmente,
  # depois verificar que no GKE que o cluster foi corretamente criado.
  # ansible-playbook -i /home/vagrant/provision_files/inventory/gcp.yml /home/vagrant/provision_files/create-gke-cluster.yml

  # comando a usar mais tarde:
  # ansible-playbook -i ${PROVISION_FILES_PATH}/inventory/gcp.yml ${PROVISION_FILES_PATH}/create-gke-cluster.yml
SHELL