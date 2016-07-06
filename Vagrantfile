# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.network "forwarded_port", guest: 80, host: 8000
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 5432, host: 5432

  config.vm.synced_folder "~/workspaces/public/geodash-server.git", "/home/vagrant/geodash-server.git"
  config.vm.synced_folder "~/workspaces/public/geodash-framework-django.git", "/home/vagrant/geodash-framework-django.git"
  config.vm.synced_folder "~/workspaces/public/geodash-base.git", "/home/vagrant/geodash-base.git"

  config.vm.provider "virtualbox" do |vb|\
      vb.gui = true
      vb.cpus = 2
      vb.memory = 4096
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/ubuntu_geodash.yml"
    ansible.host_key_checking = false
    ansible.verbose = "v"
    ansible.raw_arguments = [
      "--extra-vars=@secret.yml",
      "--extra-vars=@ansible/extra_vars/env/vagrant.yml",
      "--extra-vars=@ansible/extra_vars/os/ubuntu.yml"
    ]
  end

end
