# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/centos7"

  config.vm.hostname = "lite"
  config.vm.network :private_network, ip: "192.168.94.101"

  config.vm.synced_folder "./share", "/share"
  config.vm.synced_folder "./", "/vagrant"

  # Update CA certificates
  config.vm.provision "shell" do |s|
    s.path = "https://raw.githubusercontent.com/rurumimic/no-check-certificate/main/centos/7/update-certs.sh"
  end

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "ansible/playbook.yaml"
    ansible.vault_password_file = "ansible/vault.secret"
    ansible.limit = "all"
  end
end
