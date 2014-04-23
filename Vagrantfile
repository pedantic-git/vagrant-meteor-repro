# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "trusty-server"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"

  config.vm.network "forwarded_port", guest: 3000, host: 3000

  config.vm.provision "shell", :inline => %{
    sudo apt-get update
    sudo apt-get install -y mongodb
    curl https://install.meteor.com | sh
    echo 'export MONGO_URL=mongodb://localhost' > /etc/profile.d/mongo_url.sh
  }
end
