Using Linux with Active Directory
======================================================================================

This repository is built on the superior work of my colleagues:
* https://github.com/jannikhuels/workshop-handson-ansible
* https://github.com/jonashackt/ansible-windows-docker-springboot

The purpose of this repository is to illustrate how to connect a Linux system with Active Directory so that users and groups are configured in AD exclusively.


# Setup: The demo environment

The project uses [Vagrant](https://www.vagrantup.com/) and [Ansible](https://docs.ansible.com) to provision a Windows Domain Controller. Then, a Linux system is created and linked to Active Directory.


#### a) Installation of VirtualBox, Packer, Vagrant, Ansible

###### on Mac via [brew](https://brew.sh/index_de.html)
* `brew install packer`
* `brew cask install virtualbox`
* `brew cask install vagrant`
* `brew install python`
* `pip install ansible`

###### on Windows via [chocolatey](https://chocolatey.org/)
* `choco install packer`
* `choco install virtualbox`
* `choco install vagrant`
* TBD: Ansible

###### on Linux - use your package manager

#### Windows Server 2016 Vagrant Box:

[How to legally create a Windows VagrantBox](https://blog.codecentric.de/en/2017/04/ansible-docker-windows-containers-spring-boot/).


###### 1.) Download ISO

https://www.microsoft.com/de-de/evalcenter/evaluate-windows-server-2016

Copy file into directory `windows-domain-controller`.

###### 2.) Create Vagrant Box with Packer
```
cd windows-domain-controller
```

```
packer build -var iso_url=14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_EN-US.ISO -var iso_checksum=70721288bbcdfe3239d8f8c0fae55f1f windows_2016_domain_controller.json
```

```
vagrant box add --name windows_2016_domain_controller_virtualbox  windows_2016_domain_controller_virtualbox.box
```

```
vagrant up
```

Check Windows Connectivity:

```
ansible all -m win_ping -i inventoryStuttgart
```
