Using Linux With Active Directory
======================================================================================

This repository is built on the superior work of my colleagues:
* https://github.com/jannikhuels/workshop-handson-ansible
* https://github.com/jonashackt/ansible-windows-docker-springboot

The purpose of this repository is to illustrate how to connect a Linux system with Active Directory so that users and groups are configured in AD exclusively.

The project uses [Vagrant](https://www.vagrantup.com/) and [Ansible](https://docs.ansible.com) to provision a Windows Domain Controller. Then, a Linux system is created and linked to Active Directory.

## Tools: VirtualBox, Packer, Vagrant, Ansible

#### Mac via [brew](https://brew.sh/index_de.html)
* `brew install packer`
* `brew cask install virtualbox`
* `brew cask install vagrant`
* `brew install python`
* `pip install ansible`

#### Linux - use your package manager

#### Windows
Running Ansible from a Windows machine is beyond the scope of this document. Check [Jonas' Ansible on Linux Workshop](https://github.com/jonashackt/ansible-linux-windows-workshop) if you need to use this (German only though).

## Provisioning The Demo Data Center

### Domain Controller: Windows Server 2016 Vagrant Box

[How to legally create a Windows VagrantBox](https://blog.codecentric.de/en/2017/04/ansible-docker-windows-containers-spring-boot/).


#### 1. Download ISO

https://www.microsoft.com/de-de/evalcenter/evaluate-windows-server-2016

Copy file into directory `windows-domain-controller`.

#### 2. Create Vagrant Box With Packer
```
cd windows-domain_controller
```

```
packer build -var iso_url=14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_EN-US.ISO -var iso_checksum=70721288bbcdfe3239d8f8c0fae55f1f windows_2016_domain_controller.json
```

#### 3. Start Vagrant Box
```
vagrant box add --name windows_2016_domain_controller_virtualbox  windows_2016_domain_controller_virtualbox.box
```

```
vagrant up
```

#### 4. Check Connectivity:

```
cd ..
```

```
ansible domaincontroller -m win_ping -i ansible/inventory
```

On Mac, if you get this error:
```
objc[24289]: +[__NSPlaceholderDate initialize] may have been in progress in another thread when fork() was called.
objc[24289]: +[__NSPlaceholderDate initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead. Set a breakpoint on objc_initializeAfterForkError to debug.
```

then [disable the Fork Safety](https://github.com/ansible/ansible/issues/31869)
```
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

### Client: CentOS Linux Vagrant Box

#### 1. Start Vagrant Box

The centos client only has to be started. It will automatically download the Vagrant Box if necessary.

```
cd centos-client
```

```
vagrant up
```

#### 2. Check Connectivity

```
cd ..
```

Ping machine

```
ansible centos -m ping -i ansible/inventory
```

## Setting Up The Domain

### Creating The Domain Controller

The whole domain setup is automated by this Ansible playbook:

```
ansible-playbook ansible/domain_controller.yml -i ansible/inventory
```

### Creating Domain Users & Groups

```
ansible-playbook ansible/domain_groups_and_users.yml -i ansible/inventory
```

## Connecting Linux To The Domain

There are two alternative scripts to add the linux system to the domain. One is using SSSD, the other Samba winbind. Choose your preferred option.

#### SSSD

```
ansible-playbook ansible/add_linux_to_domain_with_sssd.yml -i ansible/inventory
```

#### winbind

```
ansible-playbook ansible/add_linux_to_domain_with_winbind.yml -i ansible/inventory
```

### A note on DNS

Connecting the Linux system to the Active Directory requires a stable DNS configuration.
This is actually the most fragile part in the setup.

The Linux system must know itself by the future FQDN already when requesting then
join from the Domain Server. To accomplish this, the `etc/hosts` file must contain
an entry for the FQDN, as the Domain Controller will only add the system to its
own DNS database after the join.

```
172.16.2.51 centos-client.linuxdc.vagrant centos-client
```

Also, the join has a high likelihood to fail in the Vagrant environment because
of NAT IP confusion. VirtualBox and Vagrant create an environment where all
boxes share the same IP address on the first network interface - the NAT connection
to the outside world. If the Linux box does receives multiple IP address responses
when querying the Domain Controller address, errors during the join can happen.
In this case, disable one of the network interfaces in the AD DNS configuration
can help.

```TODO: Screenshot of AD screen where DNS listening can be configured```

## Validating The Setup

At this time, you can log into the Linux box with the Windows users.

The winbind variant resolves the domain automatically, so simple usernames (`bob`) will work. The ssd variant expects full names (`bob@linuxdc.vagrant`). The password is `A1+bcde`.

## Other Ansible Helper Scripts

* **keyboard_german_mac.yml:** Configures the keyboard of the Linux VMs to a German Mac layout.  
* **reset-linux-vm.yml:** Destroys the Linux VMs and cleans up the ssh known_hosts file.
