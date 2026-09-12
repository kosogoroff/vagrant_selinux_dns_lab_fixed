# -*- mode: ruby -*-
# vi: set ft=ruby :

# Имя бокса можно переопределить через переменную окружения:
#   VAGRANT_BOX=almalinux9-stand vagrant up --provider=libvirt
BOX_NAME = ENV['VAGRANT_BOX'] || 'almalinux/9'

Vagrant.configure(2) do |config|
  config.vm.box = BOX_NAME
#  config.vm.box_version = "9.4.20240805" if BOX_NAME == 'almalinux/9'

  config.vm.provision "ansible" do |ansible|
    #ansible.verbose = "vvv"
    ansible.playbook = "provisioning/playbook.yml"
    ansible.become = "true"
  end

  # --- VirtualBox ---
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  # --- libvirt/KVM ---
  config.vm.provider "libvirt" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  config.vm.define "ns01" do |ns01|
    ns01.vm.synced_folder ".", "/vagrant", disabled: true
    ns01.vm.network "private_network",
      ip: "192.168.50.10",
      virtualbox__intnet: "dns",
      libvirt__network_name: "dns",
      libvirt__host_ip: "192.168.50.1",
      libvirt__netmask: "255.255.255.0"
    ns01.vm.hostname = "ns01"
  end

  config.vm.define "client" do |client|
    client.vm.synced_folder ".", "/vagrant", disabled: true
    client.vm.network "private_network",
      ip: "192.168.50.15",
      virtualbox__intnet: "dns",
      libvirt__network_name: "dns",
      libvirt__host_ip: "192.168.50.1",
      libvirt__netmask: "255.255.255.0"
    client.vm.hostname = "client"
  end
end
