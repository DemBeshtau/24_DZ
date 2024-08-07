# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :vm_name => "inetRouter",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "router-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: "router-net"},
                   {ip: "192.168.56.10", adapter: 8},
                ]
  },
  
  :centralRouter => {
        :box_name => "centos/7",
        :vm_name => "centralRouter",
        :net => [
                   {adapter:  2, auto_config: false, virtualbox__intnet: "router-net"},
                   {adapter:  3, auto_config: false, virtualbox__intnet: "router-net"},
                   {ip: "192.168.255.9", adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {ip: "192.168.56.11", adapter: 8},
                ]
  },
  
  :office1Router => {
        :box_name => "centos/7",
        :vm_name => "office1Router",
        :net => [
                    {ip: "192.168.255.10", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                    {adapter: 3, auto_config: false, virtualbox__intnet: "vlan1"},
                    {adapter: 4, auto_config: false, virtualbox__intnet: "vlan1"},
                    {adapter: 5, auto_config: false, virtualbox__intnet: "vlan2"},
                    {adapter: 6, auto_config: false, virtualbox__intnet: "vlan2"},
                    {ip: "192.168.56.20", adapter: 8},
                ]
  },
  
  :testClient1 => {
    :box_name => "centos/7",
    :vm_name => "testClient1",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.21", adapter: 8},  
            ]
},

:testServer1 => {
    :box_name => "centos/7",
    :vm_name => "testServer1",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.22", adapter: 8},  
            ]
},

:testClient2 => {
    :box_name => "ubuntu/jammy64",
    :vm_name => "testClient2",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.31", adapter: 8},  
            ]
},

:testServer2 => {
    :box_name => "ubuntu/jammy64",
    :vm_name => "testServer2",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.32", adapter: 8},  
            ]
},
}

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxconfig[:vm_name]
        
        box.vm.provider "virtualbox" do |v|
          v.gui = false
          v.memory = 512
          v.cpus = 1
        end

        if boxconfig[:box_name] == "centos/7"
          box.vm.provision "shell", inline: <<-SHELL
            sudo sed -i 's/\PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            sudo systemctl restart sshd
            sed -i 's/mirrorlist=/#mirrorlist=/g' /etc/yum.repos.d/CentOS-Base.repo
            sed -i 's/#baseurl/baseurl/g' /etc/yum.repos.d/CentOS-Base.repo
            sed -i 's/mirror.centos.org/vault.centos.org/g' /etc/yum.repos.d/CentOS-Base.repo
          SHELL
        end
        
        if boxconfig[:box_name] == "ubuntu/jammy64"
          box.vm.provision "shell", inline: <<-SHELL
            sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
            sudo systemctl restart sshd
          SHELL
        end

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", **ipconf
        end
        #boxconfig[:net].each do |ipconf|
        #  if ipconf[:ip]
        #    box.vm.network "private_network", ip: ipconf[:ip], adapter: ipconf[:adapter], netmask: ipconf[:netmask], virtualbox__intnet: ipconf[:virtualbox__intnet], auto_config: ipconf.fetch(:auto_config, true)
        #  else
        #    box.vm.network "private_network", adapter: ipconf[:adapter], virtualbox__intnet: ipconf[:virtualbox__intnet], auto_config: ipconf[:auto_config]
        #  end
        #end
        
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
  
    end
  end
end

