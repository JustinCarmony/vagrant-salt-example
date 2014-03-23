# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "precise64"

  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  config.vm.provision :salt do |salt|
        salt.minion_config = "saltstack/etc/minion"
        salt.run_highstate = true
        
        salt.install_type = "git"
        salt.install_args = "v2014.1.0"
        salt.verbose = true
    end
end
