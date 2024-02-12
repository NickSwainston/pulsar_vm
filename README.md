# pulsar_vm
Scripts for creating virtual machines for pulsar software

## Required software

- [Vagrant](https://www.vagrantup.com/)
- [Packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli)
- [VirtualBox](https://www.virtualbox.org/)

# Making the virtualbox VM

## Method


Created a base virtaulbox ova file with

```
packer build cinnamon_mint_box_from_iso.json
```

Then manually going through the install steps and making to make a pulsar user with the password pulsar.
Wait and it will perform a few more terminal commands to set up ssh.

Once that is completed you can package it into a box with

```
vagrant package --base linuxmint-21.3-cinnamon --output linuxmint-21.3-cinnamon.box
```

You can add this basebox to your local vagrant so it can be used by your `Vagrantfile` with the following command:

```
vagrant box add --name linuxmint-21.3-cinnamon linuxmint-21.3-cinnamon.box
```

This base box is then used in the `Vagrantfile` to create a VM with the following command:

```
vagrant up --provider virtualbox
```

Once it's finished provisioning (installing software) shut down the VM.
Open VirtualBox, find the VM you just created, and export the VM as an `.ova` file.


## Method that should work that doesn't

Once it's finished provisioning (installing software) you can then run

```
vagrant package --output pulsar_vm.ova
```

Which will make a `.ova` file that can be imported into virtualbox or other virtual machine software.

# Making the QEMU VM (couldn't get it to work on linux)

Created a base virtaulbox ova file with

```
packer build cinnamon_mint_box_from_iso_qemu.json
```

Then manually going through the install steps and making to make a pulsar user with the password pulsar.
Wait and it will perform a few more terminal commands to set up ssh.

Once that is completed you can package it into a box with

```
sudo qemu-img convert -f raw -O qcow2  output-virtualbox-iso/linuxmint-21.3-cinnamon.ovf  ubuntu.qcow2
mv ubuntu.qcow2 box.img
tar cvzf linuxmint-21.3-cinnamon_qemu.box ./metadata.json ./Vagrantfile ./box.img
```

You can add this basebox to your local vagrant so it can be used by your `Vagrantfile` with the following command:

```
vagrant box add --name linuxmint-21.3-cinnamon linuxmint-21.3-cinnamon_qemu.box
```

This base box is then used in the `Vagrantfile` to create a VM with the following command:

```
vagrant up --provider qemu
```