# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"


Vagrant.configure(2) do |config|
  # ПЕРВАЯ ВИРТУАЛЬНАЯ МАШИНА
    config.vm.define "web-server" do |server|
    # имя виртуальной машины
    server.vm.box = 'vedoff/centos-7-5' 
    server.vm.provider "virtualbox" do |vb|
      vb.name = "web-server"
    end
    # hostname виртуальной машины
    server.vm.hostname = "web-server"
    # настройки сети
    server.vm.network "private_network", ip: "192.168.56.161"
    server.vm.synced_folder ".", "/vagrant",  
          type: "rsync",
          rsync_auto: "true",
          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #server.vm.provision "shell", path: "provision/prepare-host.sh"
    end

  # ВТОРАЯ ВИРТУАЛЬНАЯ МАШИНА
    config.vm.define "log-server" do |client|
    # имя виртуальной машины
    client.vm.box = 'centos/7' 
    client.vm.provider "virtualbox" do |vbc|
      vbc.name = "log-server"
    end
    # hostname виртуальной машины
    client.vm.hostname = "log-server"
    # настройки сети
    client.vm.network "private_network", ip: "192.168.56.162"
    client.vm.synced_folder ".", "/vagrant",  
          type: "rsync",
          rsync_auto: "true",
          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #client.vm.provision "shell", path: "provision/prepare-host.sh"
# ssh-pub add in server
  config.vm.provision "shell", inline: <<-SHELL
  cat /vagrant/provision/ssh/vagrant-key.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL

    end
end

