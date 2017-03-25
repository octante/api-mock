# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provider :virtualbox do |v|
    v.name = "apimock"
    v.memory = 2048
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.hostname = "apimock"
  config.vm.network :private_network, ip: "10.0.0.65"
  config.vm.synced_folder "src", "/var/app", type: "nfs", nfs_version: "3"

  # Set the name of the VM. See: http://stackoverflow.com/a/17864388/100134
  config.vm.define :apimock do |apimock|
  end

  # Ansible provisioner.
  config.vm.provision "ansible" do |ansible|
    #ansible.verbose = "v"
    ansible.playbook = "provisioning/playbook.yml"
    ansible.inventory_path = "provisioning/inventory"
    ansible.sudo = true
  end

end
