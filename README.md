vagrant-salt-example
====================

Because the [documentation](https://docs.vagrantup.com/v2/provisioning/salt.html) 
for using [Vagrant](https://www.vagrantup.com/) & SaltStack's [Salt](http://www.saltstack.com/)
assumes some basic knowledge of how Salt works, this repository is to show the 
most basic setup for using Vagrant & Salt.

You can just copy the files and get started, or you can read below for a little
explanation of how & why it has been setup the way it has.

## Basic Overview

Vagrant is a tool for automating virtual machines for development purposes. 
Vagrant will connect to a "provider" such as [VirtualBox](https://www.virtualbox.org/)
and create a virtual machine. It will mount folders, setup networking, and
anything else the server needs. Then it can use a "provisioner" like Salt to
install packages, edit configuration files, and start services.

A common scenario is for a "LAMP" stack. Vagrant will create an Ubuntu 12.04 LTS
virtual machine, mount your code as a shared folder, and give it an IP address. 
Vagrant will then call the Salt provider, which will install Salt & run it. Salt
will install Apache, PHP, and MySQL, set their configuration files, and then 
start their services.

### Tech Requirements

This example will need:

 - Vagrant 1.3 or higher
 - VirtualBox 4.3 or higher

### Masterless Mode

Salt traditionally runs in a Master & Minion configuration for managing multiple
servers. Since most vagrant projects just use one virtual machine, having a 
salt-master is typically not needed, so we'll run in masterless mode with only
having a salt-minion on the VM.

## Setup

### Vagrantfile

Lets go ahead and explain how we've setup our project. Lets take a look at the
``Vagrantfile`` ([view file](Vagrantfile)):

```ruby
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Vagrant Box
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # Salt Provisioner
  config.vm.provision :salt do |salt|
    # Relative location of configuration file to use for minion
    # since we need to tell our minion to run in masterless mode
    salt.minion_config = "saltstack/etc/minion"

    # On provision, run state.highstate (which installs packages, services, etc).
    # Highstate basicly means "compare the VMs current machine state against 
    # what it should be and make changes if necessary".
    salt.run_highstate = true
    
    # What version of salt to install, and from where.
    # Because by default it will install the latest, its better to explicitly
    # choose when to upgrade what version of salt to use.

    # I also prefer to install from git so I can specify a version.
    salt.install_type = "git"
    salt.install_args = "v2014.1.0"

    # Run in verbose mode, so it will output all debug info to the console.
    # This is nice to have when you are testing things out. Once you know they
    # work well you can comment this line out.
    salt.verbose = true
  end
end
```

We've stripped out everything but what you absolutely need. We declare what base
box we want to use with ``config.vm.box`` and ``config.vm.box_url``. We're using 
precise64 which is Ubuntu 12.04 LTS.

Then we configure salt with the ``config.vm.provision :salt do |salt|`` block. 
For masterless mode you will want four config values:

  - ``salt.minion_config`` - Salt minions by defaut look for a master, so we 
    want to override that behavior. Look at ``saltstack/etc/minion`` ([view file](saltstack/etc/minion))
    to see what settings you need there.
  - ``salt.run_highstate`` - This instructs ``vagrant-salt`` to run highstate
    on the VM on provision. 
  - ``salt.install_type`` - You can choose betwen installing your VM's latest
    distribution (``dist``) or from git (``git``). You must install from git to
    pick which version you wish to install.
  _ ``salt.install_args`` - You can specify which branch or tag to install from
    git. I believe you always should be able to ``vagrant destroy`` and 
    ``vagrant up`` your instance at any time. Incase the latest distribution 
    changes for your OS, its best to explicitly declare which version to use. 
    You can than always update to a newer version by changing this value to the
    new version you want.

Thats it! You can checkout the [Vagrant Documentation for Salt](https://docs.vagrantup.com/v2/provisioning/salt.html)
for other parameters you can use in the ``Vagrantfile``, but these are the core
four you'll want/need to start with.

### SaltStack folder structure

While this isn't the required way to organize your salt files, it is my personal
preferred way of organizing them. I keep all of my salt related files and 
directories in the ``saltstack/`` folder. Inside I'll have 3 directories:

  - ``saltstack/etc/`` - Directory for salt configation files, here is where I 
    like to keep my minion config (master configs too if using a Master/Minion
    setup).
  - ``saltstack/pillar/`` - Directory for Salt's ``pillar`` data ([see salt's pillar docs](http://salt.readthedocs.org/en/latest/topics/pillar/)). 
    You will likely not need this when first starting out with Vagrant & Salt.

  - ``saltstack/salt/`` - Main directory for your ``top.sls`` file and other
    salt state files & files for Salt to use.

### Minion Config

The minion config is very simple, which is located in ``saltstack/etc/minion``
([see file](saltstack/etc/minion)):

```yaml
## Look locally for files
file_client: local

## Where your salt states & files are located
file_roots:
  base:
    - /vagrant/saltstack/salt

## Where your Pillar data is located (see README.md for more details)
pillar_roots:
  base:
    - /vagrant/saltstack/pillar
```

What each section means:

```yaml
file_client: local
```

Instead of connecting to a master to look for files, look at the local file
system.

```yaml
file_roots:
  base:
    - /vagrant/saltstack/salt
```

The default location salt will look at is ``/srv/salt``. Vagrant will make a 
shared directory to your main Vagrant folder (where ever Vagrantfile is located) 
and share that on the Guest VM at ``/vagrant``. You could copy your files to
``/srv/salt`` or make a symlink, but it is just easier to point the minion to 
look at the shared folder.

```yaml
pillar_roots:
  base:
    - /vagrant/saltstack/pillar
```

Similar to ``file_roots``, by default Salt will look at ``/srv/pillar`` for it's 
pillar data. Instead lets have this point to ``/vagrant/saltstack/pillar``.

#### Whats ``base`` stand for?

Salt can be configured for different environments, i.e. ``dev``, ``stage``, 
``test``, ``prod``, etc. The default one is ``base``. If you want to learn
more about this you can read up on it [at the docs](http://salt.readthedocs.org/en/latest/ref/configuration/minion.html?highlight=environments).

## Top.sls

Every salt environment that uses states (which a state is how salt installs 
packages, configures files, etc) has a ``top.sls`` file in the ``file_roots``
directory. Because you're only managing this one Vagrant VM instead of several
servers, the ``top.sls`` file is very straight forward (but still required).

File ``saltstack/salt/top.sls``:

```yaml 
base:
    '*':
        - common
```

We have ``base`` because that is the default. Next comes the ``'*':`` because 
we want to apply some state files to all the servers. Since you only have one
VM, ``*`` is a safe match regardless of what your VM is named.

``- common`` says I want to include the set of states stored in either
``saltstack/salt/common.sls`` file or ``saltstack/salt/common/init.sls`` file.

If you added another set of states in another file like ``saltstack/salt/apache.sls``
you would have a ``saltstack/salt/top.sls`` file that looked like this:

```yaml 
base:
    '*':
        - common
        - apache
```

## Using Vagrant & Salt

Now that you have your project setup, you can call ``vagrant up`` to provision
your VM, install Salt, and have it run ``salt.highstate``. If it is already
running, you can just use ``vagrant provision`` to re-run the highstate.

From here, everything in the [salt documentation on Salt States](http://salt.readthedocs.org/en/latest/topics/tutorials/starting_states.html)
should work for you. The [list of available salt states](http://salt.readthedocs.org/en/latest/ref/states/all/index.html) 
is really helpful.

## Debugging

Sometimes you'll want to debug your salt states on your VM. The easiest way to 
do that is to ``vagrant ssh`` and run salt-call:

```bash
vagrant ssh
sudo su
salt-call state.highstate
```

## Getting Help

If you have any questions, please join the ``#salt`` channel on 
``irc.freenode.net``. They are extremely friendly there, and there is no such
thing as a "dumb question." You can see just how awesome the Salt community is!

If you have any questions or problems, please open an Issue on this GitHub 
repository. We'll do our best to answer any questions and update the docs to
help make things more clear.

Thanks for being awesome & trying out Vagrant & Salt.
