# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'socket'

REQUIRED_VAGRANT_API_VERSION_2 = "2"
BOX_NAME = ENV['BOX_NAME'] || "ubuntu"
BOX_URI = ENV['BOX_URI'] || "http://files.vagrantup.com/precise64.box"

Vagrant::Config.run do |config|

  # Box configuration
  config.vm.box = BOX_NAME
  config.vm.box_url = BOX_URI

  # Expose ports to appear as local to HOST
  # By default, 22 is exposed as 2222 for Vagrant
  ports_to_open = []

  ports_to_open << 3000 << 3000 # Express
  ports_to_open << 4000 << 4000 # Geddy
  ports_to_open << 6379 << 6379 # Redis
  ports_to_open << 8087 << 8087 # Riak

  ports_to_open.each_slice(2) do |guest_port, host_port|
    config.vm.forward_port guest_port, host_port
  end

  config.vm.boot_mode = :headless

  pkg_cmd = ""

  pkg_cmd << "apt-get update; "

  pkg_cmd << "echo '*** Fixing bugs in Ubunutu 12.04.'; "
  pkg_cmd << "/usr/share/debconf/fix_db.pl; "
  pkg_cmd << "dpkg-reconfigure dictionaries-common; "

  # Node PPA
  pkg_cmd << "apt-get install -y python-software-properties; "
  pkg_cmd << "add-apt-repository -y ppa:chris-lea/node.js; "

  # Basho dep repo
  pkg_cmd << "curl http://apt.basho.com/gpg/basho.apt.key | apt-key add -; "
  pkg_cmd << "bash -c \"echo deb http://apt.basho.com $(lsb_release -sc) main > /etc/apt/sources.list.d/basho.list\"; "

  pkg_cmd << "apt-get update; "

  pkg_cmd << "apt-get install -y --force-yes git build-essential nodejs riak redis-server; "

  pkg_cmd << "npm install -g supervisor express geddy; "

  config.vm.provision :shell, :inline => pkg_cmd
end

# Providers were added on Vagrant >= 1.1.0
Vagrant::VERSION >= "1.1.0" and Vagrant.configure(REQUIRED_VAGRANT_API_VERSION_2) do |config|
  config.vm.provider :virtualbox do |vb, override|
    config.vm.box = BOX_NAME
    config.vm.box_url = BOX_URI
    vb.boot_mode = :gui
    vb.customize ["modifyvm", :id, "--memory", 1024]
    vb.customize ["modifyvm", :id, "--cpus", 3]
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
    vb.customize ["modifyvm", :id, "--vram", 12]
    override.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.synced_folder "../", "/workspace"
  end
end

