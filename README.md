# Using Ansible with Vagrant

This lab assumes that you already have Ansible 2.0, Vagrant 1.8.1 and VirtualBox 4 or newer installed on your machine. Installation instructions are easy to find online and out of scope of this lab.

## Example 1

In this example we will create a basic Vagrant VM and learn how to manually configure Ansible to connect to it.

First cd into example-01 directory.

The most basic Vagrantfile is already provided, but you can easily generate one with this command:

    vagrant init -m ubuntu/trusty64

Start the VM with command:

    vagrant up
