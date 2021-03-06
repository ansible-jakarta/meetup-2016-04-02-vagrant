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

## Example 2

In this example we will create multiple VMs with single Vagrantfile, explore a few more ansible.cfg options, and simplify our inventory file. We'll start example-02 lab with same files we had in example-01 and make few improvements.

First let's change our Vagrantfile, so it starts three VMs instead of one. We will also make our VM creatioin a bit more efficient by using linked clone feature:

    Vagrant.configure(2) do |config|
      config.vm.provider "virtualbox" do |v|
        v.linked_clone = true if Vagrant::VERSION =~ /^1.8/
      end

      config.vm.define :vm1 do |vm1|
        vm1.vm.box = "ubuntu/trusty64"
      end

      config.vm.define :vm2 do |vm2|
        vm2.vm.box = "ubuntu/trusty64"
      end

      config.vm.define :vm3 do |vm3|
        vm3.vm.box = "ubuntu/trusty64"
      end
    end

We can start all three VMs with the same command:

    vagrant up

Just like we did in example-01, we can list SSH configuration of the VMs and use that information to create new Ansible inventory file for three VMs.

Modify new inventory template:

    vm1 ansible_ssh_host=HOSTNAME ansible_ssh_port=PORT ansible_ssh_user='USER' ansible_ssh_private_key_file='IDENTITYFILE'
    vm2 ansible_ssh_host=HOSTNAME ansible_ssh_port=PORT ansible_ssh_user='USER' ansible_ssh_private_key_file='IDENTITYFILE'
    vm3 ansible_ssh_host=HOSTNAME ansible_ssh_port=PORT ansible_ssh_user='USER' ansible_ssh_private_key_file='IDENTITYFILE'

with output of `vagrant ssh-config` command, to get something like this:

    vm1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/sergei/workspace/meetup-2016-04-02-vagrant/example-02/.vagrant/machines/vm1/virtualbox/private_key'
    vm2 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/sergei/workspace/meetup-2016-04-02-vagrant/example-02/.vagrant/machines/vm2/virtualbox/private_key'
    vm3 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2202 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/sergei/workspace/meetup-2016-04-02-vagrant/example-02/.vagrant/machines/vm3/virtualbox/private_key'

We can now connect to all three VMs simultaneously. Confirm it with command:

    ansible -i inventory -m ping all

Ansible gives us ability to store commonly used defaults in the `ansible.cfg` and ability to have a different configuration file for each working directory. We can use this to:

 1. Simplify our inventory file by moving `ansible_ssh_user` value into `ansible.cfg` file
 2. Reduce amount of typing we do when executing `ansible` or `ansible-playbook` commands

Modify `ansible.cfg` so it looks like this:

    [defaults]
    inventory = inventory
    remote_user = vagrant
    host_key_checking = False

Delete all occurances of `ansible_ssh_user='vagrant'` from the inventory file.

We can stil reach all VMs with Ansible, but without having to specify inventory file in command line:

    ansible -m ping all

## Example 3

In this example we will use Vagrant's provisioning capability and integrate it with Ansible.

For more information on how to use Ansible provisioner see these links:
 - [https://www.vagrantup.com/docs/provisioning/ansible_intro.html]
 - [https://www.vagrantup.com/docs/provisioning/ansible.html]
 - [https://www.vagrantup.com/docs/provisioning/ansible_common.html]

Since Vagrant's Ansible provisioner expects a playbook, we'll create a very simple one in `hello.yml`:

    ---
    - hosts: all
      tasks:
        - debug:
            msg: "Hello World!"

We will also copy our `ansible.cfg` file from example-01 to disable host key checking.

Then we'll modify our single VM Vagrantfile from example-01 to include provisioning section like this:

    Vagrant.configure(2) do |config|
      config.vm.box = "ubuntu/trusty64"

      config.vm.provision :ansible do |ansible|
        ansible.playbook = "hello.yml"
        ansible.verbose = "v"
      end
    end

Now run `vagrant up` command and in addition to the usual Vagrant boot infromation you will see something like this:

    PYTHONUNBUFFERED=1 ANSIBLE_FORCE_COLOR=true ANSIBLE_HOST_KEY_CHECKING=false ANSIBLE_SSH_ARGS='-o UserKnownHostsFile=/dev/null -o IdentitiesOnly=yes -o ControlMaster=auto -o ControlPersist=60s' ansible-playbook --connection=ssh --timeout=30 --limit='default' --inventory-file=/Users/sergei/workspace/meetup-2016-04-02-vagrant/example-03/.vagrant/provisioners/ansible/inventory -v hello.yml
    Using /Users/sergei/workspace/meetup-2016-04-02-vagrant/example-03/ansible.cfg as config file

    PLAY [all] *********************************************************************

    TASK [setup] *******************************************************************
    ok: [default]

    TASK [debug] *******************************************************************
    ok: [default] => {
        "msg": "Hello World!"
    }

    PLAY RECAP *********************************************************************
    default                    : ok=2    changed=0    unreachable=0    failed=0

When Vagrant boots VM for the first time, or when you execute command `vagrant provision`, Vagrant will effectively run the following command:

    ansible-playbook --limit 'default' -i .vagrant/provisioners/ansible/inventory -v hello.yml

The advantage of this approach is that we didn't have to manually create our own inventory, Vagrant creates and updates one dynamically. Plus we can now provision whatever software we want onto the VM with Ansible as part of VM creation.

We can still run ad-hoc commands or other playbooks, similar to the way we did in previous examples, but now using using Vagrant provided inventory `.vagrant/provisioners/ansible/inventory`.

Try it out by gathering facts from the VM with this command:

    ansible -i .vagrant/provisioners/ansible/inventory -m setup default
