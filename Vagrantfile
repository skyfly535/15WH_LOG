 # -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :logS => {                      # объявление переменных параметров для машины лога
          :box_name => "centos/7",
          :box_version => "2004.01",
          :ip_addr => '192.168.1.72'
    },
  
    :logW => {                      # объявление переменных параметров для машины Web
          :box_name => "centos/7",
          :box_version => "2004.01",
          :ip_addr => '192.168.1.73'
    }
  }
  
  Vagrant.configure("2") do |config|
  
    MACHINES.each do |boxname, boxconfig| # разворачиваем машины в цикле
  
        config.vm.define boxname do |box|
            # применение параметров машин 
            box.vm.box = boxconfig[:box_name]
            box.vm.box_version = boxconfig[:box_version]
            box.vm.host_name = boxname.to_s
  
            box.vm.network "public_network", ip: boxconfig[:ip_addr] # сеть внешняя
  
            box.vm.provider :virtualbox do |vb|
              vb.customize ["modifyvm", :id, "--memory", "512"]
            end

            box.vm.provision "ansible" do |ansible|  # вызываем playbook для развертывания нужной инфраструктуры на машинах
              ansible.verbose = "v"
              ansible.playbook = "nginx.yml" # файл playbook
            end
        end
    end
  end

