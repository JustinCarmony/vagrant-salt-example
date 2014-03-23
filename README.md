vagrant-salt-example
====================

Because the [documentation](https://docs.vagrantup.com/v2/provisioning/salt.html) 
for using [Vagrant](https://www.vagrantup.com/) & SaltStack's [Salt](http://www.saltstack.com/)
assumes some basic knowledge of how Salt works, this repository is to show the 
most basic setup for using Vagrant & Salt.

You can just copy the files and get started, or you can read below for a little
explination of how & why it has been setup the way it has.

## Basic Overview

Vagrant is a tool for auotmating virtual machines for development purposes. 
Vagrant will connect to a "provider" such as [VirtualBox](https://www.virtualbox.org/)
and create a virtual machine. It will mount folders, setup networking, and
anything else the server needs. Then it can use a "provisioner" like Salt to
install packages, edit configuration files, and start services.

A common scenario is for a "LAMP" stack. Vagrant will create an Ubuntu 12.04 LTS
virtual machien, mount your code as a shared folder, and give it an IP address. 
Vagrant will then call the Salt provider, which will install Salt & run it. Salt
will install Apache, PHP, and MySQL, set their configuration files, and then 
start their services.

### Tech Requirements

This example will need:

 - Vagrant 1.3 or higher
 - VirtualBox 4.3 or higher

### Masterless Mode

Salt traditionally runs in a Master & Minion configuration for managing multiple
servers. Since most vagrant projects just use one virutla machine, having a 
salt-master is typically not needed, so we'll run in masterless mode with only
having a salt-minion on the VM.

## Setup

Lets go ahead and explain how we've setup our project. Lets take a look at the
``Vagrantfile`` ([view file](Vagrantfile)):

