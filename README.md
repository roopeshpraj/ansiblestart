Best way to start learning ansible is by spinning up multiple VMs using Vagrant. Entire exercise is done using AWS VM (free tier) - OS used is Ubuntu Server 16.04 LTS. 

#install virtual box 
sudo apt-get update
sudo apt-get install virtualbox
sudo apt-get install virtualbox-dkms

sudo apt-get install vagrant (?)

#vagrant box add debian/jessie64 https://atlas.hashicorp.com/debian/boxes/jessie64
vagrant box add precise32 http://files.vagrantup.com/precise32.box

$ mkdir vagrant_project
$ cd vagrant_project
$ vagrant init

config.vm.box = "precise32"
#config.vm.box = "debian/jessie64

mkdir test
cd test
touch VagrantFile

'''
# Defines our Vagrant environment
#
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # create mgmt node
  config.vm.define :mgmt do |mgmt_config|
      mgmt_config.vm.box = "ubuntu/trusty64"
      mgmt_config.vm.hostname = "mgmt"
      mgmt_config.vm.network :private_network, ip: "10.0.15.10"
      mgmt_config.vm.provider "virtualbox" do |vb|
        vb.memory = "256"
      end
      mgmt_config.vm.provision :shell, path: "bootstrap-mgmt.sh"
  end

  # create load balancer
  config.vm.define :lb do |lb_config|
      lb_config.vm.box = "ubuntu/trusty64"
      lb_config.vm.hostname = "lb"
      lb_config.vm.network :private_network, ip: "10.0.15.11"
      lb_config.vm.network "forwarded_port", guest: 80, host: 8080
      lb_config.vm.provider "virtualbox" do |vb|
        vb.memory = "256"
      end
  end

  # create some web servers
  # https://docs.vagrantup.com/v2/vagrantfile/tips.html
  (1..2).each do |i|
    config.vm.define "web#{i}" do |node|
        node.vm.box = "ubuntu/trusty64"
        node.vm.hostname = "web#{i}"
        node.vm.network :private_network, ip: "10.0.15.2#{i}"
        node.vm.network "forwarded_port", guest: 80, host: "808#{i}"
        node.vm.provider "virtualbox" do |vb|
          vb.memory = "256"
        end
    end
  end

end
'''
vagrant up
vagrant status
vagrant ssh
exit
vagrant destroy

Public catalogue https://atlas.hashicorp.com/boxes/search?_ga=2.190141867.1636905618.1493802182-1441353024.1491222537
Discover https://atlas.hashicorp.com/boxes/search?utf8=%E2%9C%93&sort=&provider=&q=debian

vagrant@mgmt:~/.ssh$ ssh-copy-id vagrant@lb

agrant@mgmt:~/.ssh$ ssh vagrant@lb
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/
New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Welcome to your Vagrant-built virtual machine.
Last login: Mon May  8 11:06:36 2017 from 10.0.15.10
vagrant@lb:~$ exit
logout
Connection to lb closed.
vagrant@mgmt:~/.ssh$ ssh lb
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/
New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Welcome to your Vagrant-built virtual machine.
Last login: Mon May  8 11:31:45 2017 from 10.0.15.10
vagrant@lb:~$ exit
logout


