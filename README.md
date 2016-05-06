# Using Ansible with Vagrant

This lab assumes that you already have Ansible 2.0, Vagrant 1.8.1 and VirtualBox 4 or newer installed on your machine. Installation instructions are easy to find online and out of scope of this lab.

## Example 1

In this example we will create a basic Vagrant VM and learn how to manually configure Ansible to connect to it.

First cd into example-01 directory.

The most basic Vagrantfile is already provided, but you can easily generate one with this command:

    vagrant init -m ubuntu/trusty64

Start the VM with command:

    vagrant up

By default Ansible uses SSH for remote administration. To establish SSH connection to host (or VM) we need:

 - IP/hostname
 - SSH port
 - username
 - private/public key pair (or password)

Few notes about Vagrant's way of dealing with SSH:

 - default user is `vagrant`
 - default password is `vagrant`
 - Vagrant base boxes have well known public key in `~/vagrant/.ssh/authorized_keys` to allow Vagrant to login, but it's replaced with newly generated public key on first boot, while new private key is placed into `.vagrant/machines/default/virtualbox/private_key`
 - Vagrant finds an available port in the `22XX` range to forward traffic to VM's SSH server. That port is allocated dynamically on VM boot, so don't expect it to always be 2222, especially if you have more than one Vagrant VM running on you machine.

Check SSH configuration of the VM by executing command:

    vagrant ssh-config

Sample output:

    Host default
      HostName 127.0.0.1
      User vagrant
      Port 2200
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile "/Users/sergei/workspace/meetup-2016-04-02-vagrant/example-01/.vagrant/machines/default/virtualbox/private_key"
      IdentitiesOnly yes
      LogLevel FATAL

With this information we can create an inventory file for Ansible to use. Modify the includded template by replacing placeholders with values from your VM.

Inventory template:

    HOST ansible_ssh_host=HOSTNAME ansible_ssh_port=PORT ansible_ssh_user='USER' ansible_ssh_private_key_file='IDENTITYFILE'

Sample resulting inventory:

    default ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/sergei/workspace/meetup-2016-04-02-vagrant/example-01/.vagrant/machines/default/virtualbox/private_key'
