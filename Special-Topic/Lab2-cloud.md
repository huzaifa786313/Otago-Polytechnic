# IN730 Special Topic - Network Automation
<br>

## Lab2 - Ansible Playbooks (Cloud Version)

### Requirements

- Completion of lab 1
- Azure Subscription

### Topology

<img src="Images/topologycloud.JPG">

If you stopped your virtual machine that was running your gns3server and you didnt make your tap1 and ip routes persistent then you will need to run the following commands again to recreate those

```
sudo tunctl -t tap1
sudo ifconfig tap1 192.168.1.254 netmask 255.255.255.0 up
sudo ip route add 192.168.1.0/24 via 192.168.1.254 dev tap1
sudo ip route add 192.168.2.0/30 via 192.168.1.254 dev tap1
```

## Using ansible playbooks to pull device information

create a directory to be used for backups of the routers
```
sudo mkdir ~/ ansible-backups
```

create a playbook called backup.yaml

```
sudo vim /etc/ansible/backup.yaml
```

insert the following into the backup.yaml file
```
---
  - hosts: localhost

    tasks:
            - name: Get Date/Time
              setup:
                      filter: "ansible_date_time"
                      gather_subset: "!all"

            - name: Store Date/Time
              set_fact:
                      DTG: "{{ansible_date_time.date }}"

            - name: Create Directory {{hostvars.localhost.DTG}}
              file:
                      path: /home/gns3server/ansible-backup/{{hostvars.localhost.DTG}}
                      state: directory
    run_once: true

  - hosts: network
    connection: local
    remote_user: admin
    gather_facts: false
    tasks:
            - name: backup running config
              block:
              - name:
                ios_command:
                  commands: show running-config
                register: config

              - name: save running config to backup folder
                copy:
                  content: "{{config.stdout[0]}}"
                  dest: "/home/gns3server/ansible-backup/{{hostvars.localhost.DTG}}/{{inventory_hostname}}-{{hostvars.localhost.DTG}}-config.txt"
```

now run the playbook which will run 
```
show running-config
```
and store this 
```
ansible-playbook backup.yaml
```
we will instead an application called tree which will help with displaying *REWORD*
```
sudo apt-get install tree
```
we can now use the following command to list the home directory
```
tree ~/
```
you should have an output simillar to the following

<img src="Images/treeconfig.JPG">

here we can see that ansible managed to pull configuration from the routers
## push configs i.e. motd banners etc for uniform deployments?

```
---
  - name: testbook
    hosts: network
    connection: local
    remote_user: admin
    gather_facts: false
    tasks:
            - name: configure login banner
              ios_banner:
                      banner: login
                      text: |
                              Here
                              Is
                              A
                              Test
                              Configuration
                              Banner
                      state: present
```

now lets connect to our router to see the change we made

due to issues with gns3 and cloud we need to add a few additional options to our ssh command

```
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -c 3des-cbc admin@<router IP>
```

if we now ssh onto the router we can see that ansible has configured a motd banner

<img src="Images/sshmotd.JPG">

## run a check on your config backups to make sure that they are configured the same - except for the interface ip and such

## bonus create vms? azure/openstack using ansible?

```
sudo apt install curl
```

download and install azure command line
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
sudo apt install azure-cli
```

connect to the azure command line
```
az login
```

follow the on screen prompt and input the code it provides then select your microsoft account that you used for the previous lab

after you have logged in you will be given an output that contains the information about your microsoft azure account *SOMETHING* 

- name

<img src="Images/azdetails.jpg">

when you use azure cli it will assign anything you create to your default subscription so if you have multiple subscriptions on your account we will need to set this

to change your default subscription is a quick task

```
az account set --subscription <ID>
```

after changing your subscription you wont be given a notification so you will need to verify that it has changed by using the following command
```
az account list
```

- sudo apt-get install python-pip
- pip install packaging
- pip install msrestazure
- pip install ansible[azure]

playbook that will create *NEED TO CHANGE THE VARIABLES AS THEY STILL RELATE TO GITLAB TEST DEPLOYMENT*
```
- name: Create Azure VM
  hosts: localhost
  connection: local

  vars:
   vm_offer: "UbuntuServer"
   vm_pub: "Canonical"
   vm_sku: "18.04-LTS"

   vm_size: "Standard_E2s_v3"

   az: "australiaeast"

   vm_net: "myVNet"
   vm_subnet: "mySubnet"

   vm_publicIP: "myPublicIP"
   vm_NSG: "myNSG"
   vm_NIC: "myNIC"
   vm_Name: "gitlab-test"

   resource_group: "gitlab_test"

   os_user: "azuser"
   os_pass: "AzuserP@ssw0rd"

  tasks:

  - name: Create a resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ az }}"
      tags:
        testing: testing
        delete: never

  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_net }}"
      address_prefixes: "10.0.0.0/16"

  - name: Add subnet
    azure_rm_subnet:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_subnet }}"
      address_prefix: "10.0.1.0/24"
      virtual_network: "{{ vm_net }}"

  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: "{{ resource_group }}"
      allocation_method: Static
      name: "{{ vm_publicIP }}"
      domain_name: gitlab-test
    register:  reg_publicIP

  - debug: var=reg_publicIP

  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_NSG }}"
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound
        - name: HTTP
          protocol: Tcp
          destination_port_range: 80
          access: Allow
          priority: 1002
          direction: Inbound
        - name: HTTPS
          protocol: Tcp
          destination_port_range: 443
          access: Allow
          priority: 1003
          direction: Inbound
        - name: AERO
          protocol: Tcp
          destination_port_range: 8060
          access: Allow
          priority: 1004
          direction: Inbound
        - name: UNKNOWN
          protocol: Tcp
          destination_port_range: 9094
          access: Allow
          priority: 1005
          direction: Inbound  

  - name: Create virtual network interface card
    azure_rm_networkinterface:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_NIC }}"
      virtual_network: "{{ vm_net }}"
      subnet: "{{ vm_subnet }}"
      public_ip_name: "{{ vm_publicIP }}"
      security_group: "{{ vm_NSG }}"

  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: "{{ resource_group }}"
      name: "{{ vm_Name }}"
      vm_size: "{{ vm_size }}"
      admin_username: "{{ os_user }}"
      admin_password: "{{ os_pass }}"
      ssh_password_enabled: true
      network_interfaces: "{{ vm_NIC }}"
      image:
        offer: "{{ vm_offer }}"
        publisher: "{{ vm_pub }}"
        sku: "{{ vm_sku }}"
        version: latest
```

using azure peerings so that your private ip network in 1 region can communicate with another private network in a different region i.e. AUEast w/ AUSouthEast

<br>