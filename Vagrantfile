# -*- mode: ruby -*-
# # vi: set ft=ruby :

require_relative 'override-plugin.rb'

NUM_INSTANCES = (ENV['NUM_INSTANCES'].to_i > 0 && ENV['NUM_INSTANCES'].to_i) || 1

CLOUD_CONFIG_PATH = "./user-data"

Vagrant.configure("2") do |config|
  config.vm.box = "coreos-alpha"
  config.vm.box_url = "http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_vagrant.box"

  config.ssh.private_key_path = ["#{ENV['HOME']}/.ssh/iknapp.id", "#{ENV['HOME']}/.vagrant.d/insecure_private_key"]

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://storage.core-os.net/coreos/amd64-usr/alpha/coreos_production_vagrant_vmware_fusion.box"
  end

  # Fix docker not being able to resolve private registry in VirtualBox
  config.vm.provider :virtualbox do |vb, override|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  (1..NUM_INSTANCES).each do |i|
    config.vm.define vm_name = "core-%02d" % i do |config|
      config.vm.hostname = vm_name

      ip = "172.17.8.#{i+100}"
      config.vm.network :private_network, ip: ip
      #config.vm.network :public_network, :bridge => 'en0: Wi-Fi (AirPort)'
      config.vm.network :forwarded_port, guest: 4243, host: 4243, :host_ip => "127.0.0.1"
      (49000..49900).each do |port|
        config.vm.network :forwarded_port, :host => port, :host_ip => "127.0.0.1", :guest => port
      end

      config.vm.synced_folder "#{ENV['HOME']}/.coreos", "/media/host", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']

      if File.exist?(CLOUD_CONFIG_PATH)
        config.vm.provision :file, :source => "#{CLOUD_CONFIG_PATH}", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end
    end
  end
end
