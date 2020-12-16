# Installation of Home Assistant Operating System on Centos 

Installation of Home Assistant Operating System on CentOS Stream 8 with Ansible. This might work on other RHEL based family of operating systems such as Centos 8 and Fedora

## Preparing the host machine

Download the CentOS Stream version 8 iso preferably the DVD version. 

### Downloading the ISO

Icelandic mirror for the lates version at the time of writing: http://mirrors.opensource.is/CentOS/8-stream/isos/x86_64/CentOS-Stream-8-x86_64-20201211-dvd1.iso . Other mirrors can be found here: https://www.centos.org/download/mirrors/

> If you decide to use `CentOS-Stream-8-x86_64-20201211-boot.iso` then you might have to update the source list in the installer menu with a repository URL f.ex. http://mirrors.opensource.is/CentOS/8-stream/BaseOS/x86_64/os/



### Writing the ISO file to a USB disk

General instruction can be found here https://wiki.centos.org/HowTos/InstallFromUSBkey and you can search various instruction on the internet how to write an iso to a USB disk. Please take note that the DVD iso is 9GB.



## Preparing for ansible automated install

In order to use this playbook for installing the virtual appliance you will have to get familiar with Ansible or at least aquire the minimum knowledge of creating public/private keypair to use to log into the server where you will be executing the playbook towards. 



#### Generating SSH keypair if you do not have one

For example if you are on a linux machine or a mac you can open up a terminal and generate a ssh-key like so:

```SHELL
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "me@example.com"
```

Then you will need to login to the remote server under the user that will execute playbook remotely and add the **public** ssh key.

```shell
ssh-copy-id -i ~/.ssh/id_ed25519 ansible@vm.host.tld
```



> Please note if you are on a windows machine you can do this if you install WSL2 ( Windows Subsystem for Linux ). Instruction can be found here: https://docs.microsoft.com/en-us/windows/wsl/install-win10



## Updating the Ansible playbook

There are three ansible variables that you should change in order to prepare your virtual machine on the host machine. If you are  new to Ansible I recommend that you check out Jeff Geerlings Ansible 101 series https://www.jeffgeerling.com/blog/2020/ansible-101-jeff-geerling-youtube-streaming-series



| Ansible Variable Name    | Default Value                                                | Comment                                                      |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| hassio_vm_name           | hassio.local                                                 | This is the name used for the image file and the name displayed by libvirt in both cockpit and when using the `virsh` shell |
| hassio_network_interface | bridge0                                                      | Find the network card that has an ip on the same network that you will be exposing the virtual appliance. This can be found f.ex. with `ip a` command and looking through the list of interfaces and their corresponding ip addresses. |
| hassio_vm_qcow_file      | https://github.com/home-assistant/operating-system/releases/download/5.8/hassos_ova-5.8.qcow2.xz | Be sure to double check this what the current image is and update accordingly. The url can be obtained by visiting https://www.home-assistant.io/hassio/installation/ and copy the url for the `qcow2 `virtual appliance. |



### Modifying the playbook

```yaml
---
- name: Install hassos virtual appliance as a kvm virtual machine
  hosts: homeassistant
  become: True
  vars:
    # Name of the virtual appliance
    - hassio_vm_name: "hassio.local"
    # Name of the network card that you will do a direct connection against on the host machine.
    - hassio_network_interface: "bridge0"
    # Path to the virtual appliance 
    - hassio_vm_qcow_file: "https://github.com/home-assistant/operating-system/releases/download/5.8/hassos_ova-5.8.qcow2.xz"
  tasks:
  	... removed for readability ...
```



### Modifying the inventory file

In the inventory file you should update the `vm.host.tld` hostname with the host that will be hosting the virtual appliance.

```yaml
---
all:
  hosts:
    vm.host.tld:
  vars:
    ansible_user: ansible
  children:
    homeassistant:
      hosts:
        vm.host.tld:
```



### Executing the playbook

Once you have your remote user, pubilc ssh-key and inventory set up you should be good to go. 



You can do a quick test with the ping command to see if ansible is able to log into the remote server and return you a pong response.

```shell
# Remember cd into the directory where the playbook and ansible.cfg, inventory.yml is located
ansible -m ping homeassistant 
```



If the test was succsessful you can now execute the playbook by issuing. Note `-v` is a verbose flag and you can add up to 5 v´s to increase verbosity. 

```shell
# Remember cd into the directory where the playbook and ansible.cfg, inventory.yml is located
ansible-playbook install_hassos.yaml -v 
```

If all goes well you should see a summary at the end of the execution where there will be indications about what was changed/ok and failed. If there are failures you can execute the playbook again with increased verbosity to see more details about the errors.



## Checking if the applicance is running

### Through the Cockpit Administrative interface

Centos and RHEL servers have a nice administrative interface called cockpit which you can use to examine your virtual appliance that is available at https://vm.host.tld:9090 if you have enabled it to run. You can start the web interface through command line with `systemctl enable --now cockpit.socket` if it is not running. 

### Through the command line

```shell
# List running vm's
virsh list
# List all vm's 
virsh list --all
# Poweroff vm
virsh shutdown <name or id of the vm>
# Start vm
virsh start <name or id of the vm>
# Force poweroff 
virsh destroy <name or id of the vm>
# Delete vm
virsh undefine <name or id of the vm>
# Log in to vm via console
virsh console <name or id of the vm>
```



