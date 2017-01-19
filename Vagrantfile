# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "bento/ubuntu-16.04"

  config.vm.network "forwarded_port", guest: 80, host: 8000
  config.vm.network "forwarded_port", guest: 443, host: 443
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 5432, host: 5432


  # These 3 repos are the main repos for main framework development.

  config.vm.synced_folder "~/workspaces/public/geodash-viewer.git", "/home/vagrant/geodash-viewer.git"
  config.vm.synced_folder "~/workspaces/public/geodash-base.git", "/home/vagrant/geodash-base.git"
  config.vm.synced_folder "~/workspaces/public/geodash.js.git", "/home/vagrant/geodash.js.git"

  # Uncomment the following if needed for application development

  #config.vm.synced_folder "~/workspaces/public/geodash-server.git", "/home/vagrant/geodash-server.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-framework-django.git", "/home/vagrant/geodash-framework-django.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-map-map.git", "/home/vagrant/geodash-plugin-map-map.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-editor-welcome.git", "/home/vagrant/geodash-plugin-editor-welcome.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-editor-sidebar.git", "/home/vagrant/geodash-plugin-editor-sidebar.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-main.git", "/home/vagrant/geodash-plugin-main.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-legend.git", "/home/vagrant/geodash-plugin-legend.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-filters.git", "/home/vagrant/geodash-plugin-filters.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-overlays.git", "/home/vagrant/geodash-plugin-overlays.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-navbars.git", "/home/vagrant/geodash-plugin-navbars.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-handlers.git", "/home/vagrant/geodash-plugin-handlers.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-plugin-base.git", "/home/vagrant/geodash-plugin-base.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-diff.git", "/home/vagrant/geodash-diff.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-flatten.git", "/home/vagrant/geodash-flatten.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-build-pipeline.git", "/home/vagrant/geodash-build-pipeline.git"
  #config.vm.synced_folder "~/workspaces/public/geodash-build-log.git", "/home/vagrant/geodash-build-log.git"
  #config.vm.synced_folder "~/workspaces/public/typeahead.js.git", "/home/vagrant/typeahead.js.git"

  config.vm.provider "virtualbox" do |vb|\
      vb.gui = false
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
