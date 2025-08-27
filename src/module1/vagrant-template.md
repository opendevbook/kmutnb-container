# Vagrant Template

Vagrantfile  for Vagrant
[https://developer.hashicorp.com/vagrant/docs/vagrantfile](https://developer.hashicorp.com/vagrant/docs/vagrantfile)

- Centos 9 Stream `generic/centos9s`
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "generic/centos9s"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.network "public_network"

  # config.vm.synced_folder "../data", "/vagrant_data"

  #config.vm.synced_folder ".", "/vagrant"


  config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL

  SHELL
end
```


- We can change template to use box  ``generic/ubuntu2310`` , below:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2310"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.network "public_network"

  # config.vm.synced_folder "../data", "/vagrant_data"

  #config.vm.synced_folder ".", "/vagrant"


  config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL

  SHELL
end
```

