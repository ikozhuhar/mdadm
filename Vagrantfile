# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 1
  end

  #config.vm.define "nfss" do |mdadm|
    #mdadm.vm.network "private_network", ip: "192.168.50.10", virtualbox__intnet: "net1"
    #mdadm.vm.hostname = "nfss"

    config.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.version = "2.10.8"
      ansible.playbook = "create_raid-1.yml"
    end
  end

end