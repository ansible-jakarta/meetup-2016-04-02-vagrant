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

We can now test our Ansible connectivity by gathering facts from the VM:

    ansible -i inventory -m setup default

The ansible execution might be interrupted and you might see a similar prompt in you command line:

    The authenticity of host '[127.0.0.1]:2200 ([127.0.0.1]:2200)' can't be established.
    RSA key fingerprint is f7:43:fa:26:9e:2f:33:85:f4:60:a5:8e:f8:bf:8b:b3.
    Are you sure you want to continue connecting (yes/no)?

This is an SSH security feature, warning you that the host you're about to connect to is unknown and its authenticity cannot be confirmed. Since we likely to create and destroy new VMs all the time, we don't want to save their fingerprints in our known_hosts file. We want Ansible to ignore host key checks for VMs.

An easy way to do so is by creating a local ansible.cfg file like so:

    [defaults]
    host_key_checking = False

We should now be able to connect to our VM uninterrupted.
