# IN730 Special Topic - Network Automation
<br>

## Disclaimer

vim is the text editor used in the following lab guide but you can use your own preferred text editor if you wish *SOMETHING ABOUT HAVING TO KNOW IT YOURSELF*

## Initial Setup
In the following lab we will install and configure the required components needed to **SOMETHING** gns3 and ansible **EXPAND**

## Local

## Topology

<img src="Images/topology.JPG">

## Requirements

- VM Workstation 
- Ubuntu VM
- GNS3
- Windows Machine

## Setup

Step X) Ubuntu Setup

X) Download Ubuntu 20.04.1 LTS image from here https://ubuntu.com/download/desktop

X) Launch the VM Workstation Application then proceed to setup the linux VM by going to File > New Virtual Machine > Typical > select browse from the installer disc image file(iso) option, then locate the iso file you downloaded > next > follow the onscreen instructions > finish

After creating the linux VM we now need to configure some network options in VM workstation

X) go to Edit > virtual network editior then click on the change settings option and accept the administrator promopt > set VMnet0 to bridge using the physical interface on your machine. Also select the VMnet that has the type and external connection of NAT and change its subnet ip to 192.168.0.0 with a subnet mask of 255.255.255.0

X) connect to your linux VM and open a terminal, then run the command "ip a" and note down the ip address on the ens33(ens number may vary but there will be only one) interface

Step X) GNS3 Setup

X) Sign up to gns3 https://www.gns3.com/ then proceed to download the windows version of gns3 https://www.gns3.com/software/download

GNS3 INSTALL STEPS HERE

Step X) Router Template Configuration

X) Download the image for the cisco c7200 router here 
https://github.com/samsojl1/Otago-Polytechnic/raw/master/Special-Topic/c7200/c7200-advipservicesk9-mz.122-33.SRC2.extracted.bin

X) Import the C7200 Router into gns3 by going file > new template > install an appliance from the GNS3 server > then click the dropdown for the routers section and select Cisco 7200 then click install > Install the appliance on your local computer > create a new version, call it whatever you wish > select your version from the list and click import, locate and select the c7200 bin file your downloaded earlier > next > accept the install > finish, if you click on the router icon on the left hand side you should now see your router template you installed

X) right click your newly created router template and click on the configure template option,

<img src="Images/template.JPG">
from here go to the slots tab and add "PA-GE" to Adapters slots 1 through 4 this will add 4 gigabyte interfaces to your routers when you spawn them

Step X) Basic Network

Lets create a simple network in GNS3

X) create a new blank project and call it whatever you want

X) lets add 2 of our newly created routers to the project by going to the router tab and dragging 2 on, cable these 2 routers together and configure them using a private ip 

On R1
```
end
conf t
int g1/0
ip address 192.168.1.1 255.255.255.252
no shut
```
On R2
```
end
conf t
int g1/0
ip address 192.168.1.2 255.255.255.252
no shut
```
X) Verify that R1 can ping R2 and vice versa

X) lets add a cloud to connect our virtual routers to our physical network 

After adding the cloud click on it and go to "Ethernet Interfaces" tab, then tick the "Show special Ethernet interfaces" box, click the Add all button, this will add all the interfaces from your physical machine to the cloud allowing you to connect your virtual router to it



on the interface you connected your R1 to the cloud you need to configure it with an ip in the same range as the physical interface, the ens** ip you recorded earlier
On R1 
```
end
conf t
int g2/0
ip address 192.168.0.1 255.255.255.0
no shut
```

Configure OSPF and a static default route then redistirbute that route into ospf

On R1
```
end
conf t
ip route 0.0.0.0 0.0.0.0 192.168.0.128
router ospf 1
router-id 1.1.1.1
network 192.168.0.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.3 area 0
default-information originate
```

On R2
```
end
conf t
router ospf 1
router-id 2.2.2.2
network 192.168.1.0 area 0
```

Step X) Route

On your linux vm you 

Step X) Ansible Setup

X) On your Linux VM open a terminal

X) run the command sudo apt-get install ansible and accept, this will install ansible onto your linux machine

X) cd into /etc/ansible/ this is where the ansible.cfg and hosts file exist from here you can create your playbooks

Inside the hosts file you can define your network devices and asign them to groups an example is provided inside the file by ansible

You can run either use ad-hoc commands or ansible playbooks 

Create a default route and redistribute it into ospf

disable host_key_checking
on line 62 uncomment host_key_checking = False

You should now be able to ping from your linux vm to R2

Because ansible uses SSH to deploy playbooks as well as ad-hoc commands, you will need to enable SSH onto your GNS3 Routers, a basic configuration has been provided 

``` 
end
conf t
ip domain-name ansible.com
crypto key generate rsa
1024
ip ssh version 2
username admin privilege 15 password 0 admin
line vty 0 4
login local
transport input ssh
exit
```

in the hosts file you can define your devices in a few different ways you can have them ungrouped

in the /etc/ansible/hosts file we will add the ip addresses of the devices we wish to use ansible against

```
[routers]
R2 ansible_host=192.168.1.2 ansible_network_os=ios ansible_ssh_user=admin ansible_ssh_pass=admin
R1 ansible_host=192.168.0.129 ansible_network_os=ios ansible_ssh_user=admin ansible_ssh_pass=admin
```
* The [routers] defines the name of the group this can be called whatever you wish
* R2 and R1 are the names of the hosts
* ansible_host=X.X.X.X is the ip of the host
* ansible_network_os=ios defines the network platform that the host is using
* ansible_ssh_user=admin the user account that ansible uses to connect with in this example its admin because that is what we created earlier when we setup the router configuration in gns3
* ansible_ssh_pass=admin the password of the user account that ansible is using to connect with

<br>

lets create a easy playbook to test if everything is working correctly
ansible can be a bit pedantic with its formating so here is a 

```
sudo vim /etc/ansible/test.yaml
```

and copy and paste the following 
```
---
  - name: ping
    hosts: routers
    connection: local
    gather_facts: false
    tasks:
            - ping:
```

then write quit
<br>

In order to run your ansible playbook that you have now created you need to be located in the directory that the playbook was made 

```
ansible-playbook test.yaml
```

After running that command the following output should occur: <br>

<img src="Images/playbook.JPG">
<br>

This means that ansible can successfully  *SOMETHING*

ansible all -c network_cli -u samsojl1 -k -m ping -e ansible_network_os=ios

Further reading:

ansible module list can be found here 
* https://docs.ansible.com/ansible/latest/modules/modules_by_category.html

ansible playbooks user guide can be found here 
* https://docs.ansible.com/ansible/latest/user_guide/playbooks.html






## Troubleshooting

If at some point your pings / connection stops working between your linux vm and your 
delete the cable connecting R1 and the cloud together then reconnect

# Azure

## Disclaimer

vim is the text editor used in the following lab guide but you can use your own preferred text editor if you wish *SOMETHING ABOUT HAVING TO KNOW IT YOURSELF*

Ansible / gns3server are the same *REWORD*

## Requirements
- Azure Subscription 

## Topology

<img src="Images/topologycloud.JPG">

## SIGNUP SECTION TO BE ADD

Azure portal https://portal.azure.com/ and sign in

## Resource Group

Create a resource group to store the project *REWORD* in

type "resource groups" in the search bar and click on the resource groups under services

now click add to make a new resource group

here you can name the resource group 
lets call it "ansible" for now

you can also select the region you want your resource group in, unless you require it to be in a specific region in order to do things it is instead best to choose a region that is the closest to you
lets select australia east from the drop down

now lets create our resource group
click the review + create and confirm the creation

## Network

we now need to create a virtual network that will be used *REWORD*

go to your ansible resource group and click the add button

go to the networking tab and select virtual network

Set the name for your virtual network to ansible<br>
Set the region for your virtual network to Australia East
```
Name - ansible
Region - Australia East
```
create ipv4 address space for your virutal network

we will make it a 192.168.0.0/16
```
ipv4 address space
192.168.0.0/16
```
we will now subnet our address range<br>
we will name this subnet ansible<br>
we will give this subnet an address range of 192.168.0.0/24
```
add subnet
name - ansible
subnet 192.168.0.0/24
```

Click the review + create and double check that it is all correct

If everything is correct click the create button
```
review + create
create
```
## VM 1 - Windows

type "resource groups" in the search bar and click on the resource groups under services

click on the ansible resource group that you created

click on the add button

this will take you to a page where you can choose from a large range of options of things to add to your resource group but for now we only need a virtual machine

on the left hand side select the compute option now select virtual machine, this will take you to a screen where you can create a virtual machine

because the free trial has a maximum amount of vcpus that you can have allocated per region we need to make sure we divided them correctly between the ansible/gns3 server and the gns3client machine

confirm that the resource group is ansible
```
details
Virtual machine name - gns3client
Region - (Asia Pacific) Australia East
Image - Windows 10 Pro
Azure Spot Instance - default
Size - Standard_B2s
Username - gns3client
Password - gns3clientP@ssw0rd
Confirm Password - gns3clientP@ssw0rd
Public inbound ports - default
Select inbound ports - default
Licensing - check
```
```
disks
Leave as default
```
```
Networking
Virtual network - Ansible
subnet - Ansible (192.168.0.0/24)
public ip - default
NIC network security group - none
Load balancing - default
```
```
Management
leave as defaults
```
```
Advanced
leave as defaults
```
```
Tags
leave as defaults
```
```
review + create
check over and make sure you have the correct options set
```
* DNS PART TBA

## VM 2 - Ansible

type "resource groups" in the search bar and click on the resource groups under services

```
Virtual machine name - gns3server
Resource Group - ansible
Region - (Asia Pacific) Australia East
Image - Ubuntu Server 18.04 LTS
Azure Spot Instance - default
Size - Standard_E2s_v3
Authentication type - Password
Username - gns3server
Password - gns3server@ssw0rd
Confirm Password - gns3server@ssw0rd
Public inbound ports - default
Select inbound ports - default
```
```
Disks
leave as defaults
```
```
Networking
Virtual network - Ansible
subnet - Ansible (192.168.0.0/24)
public ip - default
NIC network security group - none
Load balancing - default
```
```
Management
leave as defaults
```
```
Advanced
leave as defaults
```
```
Tags
leave as defaults
```
```
review + create
check over and make sure you have the correct options set
```
* DNS PART TBA

We will configre a dns on our gns3server so that connecting to it is easier *REWORD*

go to your Ansible resource group and click on your gns3server-ip

<img src="Images/gns3dns.JPG">

From here click the configuration option on the left hand side of the screen under settings 

<img src="Images/gns3dnssettings.JPG">

under "DNS name label (optional)" 

lets set our DNS name label to
```
gns3server
```

and save it

your DNS name label will be suffixed with ".australiaeast.cloudapp.azure.com" i.e. gns3server.australiaeast.cloudapp.azure.com

* If there are multiple people working on this you may need to tweak your name by appending a number onto the end i.e. gns3server1 etc. if you had to do this note the change for future steps

Alternatively you could also instead configure a static ip and use that in place of a DNS

## gns3server

The following occurs on our gns3server VM

Before we start lets make sure that we can download the latest software *REWORD*

In order to do this run the following command
```
sudo apt-get update -y
```
<br>
First thing we need to do is install gns3 server onto our linux server so that our gns3 client can connect to it

run the following commands to install the gns3 server
```
cd /tmp
curl https://raw.githubusercontent.com/GNS3/gns3-server/master/scripts/remote-install.sh > gns3-remote-install.sh
sudo bash gns3-remote-install.sh --with-openvpn --with-iou --with-i386-repository
```

Its now time to edit our server settings so that we can connect to it using the gns3client virtual machine

Run the following command

```
ip a
```

and note the ip address of eth0, we will use this in our gns3_server.conf file

Now lets edit our gns3_server.conf file

```
sudo vim /etc/gns3/gns3_server.conf
```

make sure that the "host = " is set to the ip of eth0<br>
and that the port = variable to 3081

Your gns3_server.conf file should look like the one in the image below

<img src="Images/serverconf.JPG">

After making these changes we now need to restart gns3<br> 
use the following command
```
sudo systemctl restart gns3.service
```


Now we will create a tap interface so that we can connect our virtual network that we will create in gns3 to our physical network so that it can communicate with outside devices

First we will need to download uml-utilities which will allow us to create TAP interfaces
```
sudo apt-get install uml-utilities -y
```

Now that we have uml-utilties we can go ahead and create a TAP interface
* Do note that the TAP interface and the ip associated with it are not persistant doing it the following way so you will have to run the following commands each time you shutdown or restart your VM
```
sudo tunctl -t tap1
sudo ifconfig tap1 192.168.1.254 netmask 255.255.255.0 up
```

to allow connection to the outside we need to configure some iptable rules
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i tap1 -j ACCEPT
sudo iptables -A INPUT -i tap1 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -j ACCEPT
sudo iptables -A INPUT -i eth0 -j ACCEPT
```

Create routes<br>
we need to create routes<br>
so that traffic<br>
*SOMETHING*
```
sudo ip route add 192.168.1.0/24 via 192.168.1.254 dev tap1
sudo ip route add 192.168.2.0/30 via 192.168.1.254 dev tap1
```

## Install GNS3 Client

## <p style="text-align: center;">The following occurs on our gns3client VM

Step X) GNS3 Setup

X) Sign up to gns3 https://www.gns3.com/ then proceed to download the windows version of gns3 https://www.gns3.com/software/download

GNS3 INSTALL STEPS HERE

Edit > preferences > server

change the host to gns3server.australiaeast.cloudapp.azure.com<br>
port 3081

<img src="Images/server.JPG">

leave auth unchecked

## Router Template Configuration

Because gns3 doesn't come with any routers avaliable to use by default we need to import and configure a template for one

First we will need to download the image for our cisco 7200 here is a link to the image download
```
https://github.com/samsojl1/Otago-Polytechnic/raw/master/Special-Topic/c7200/c7200-advipservicesk9-mz.122-33.SRC2.extracted.bin
```

Now lets import the cisco 7200 into gns3
1) File 
2) New template
3) Install an appliance from the GNS3 server 
4) Next 
5) Click the dropdown for the routers section and select cisco 7200
6) Click install 
7) Install the appliance on the main server 
8) Create a new version
9) Version name "Cisco 7200"<br>
<img src="Images/c7200.JPG"> 
10) select your version from the list 
<img src="Images/c7200import.JPG"> 
11) Click import, locate and select the cisco 7200 bin file your downloaded earlier, your version should change from "Missing" to "Ready to install" as shown in the images
<img src="Images/c7200install.JPG"> 
12) Click next
13) Accept the install 
14) Finish

If you click on the router icon on the left hand side<br>
<img src="Images/routericon.JPG">  
you should now see your router template you installed<br>
<img src="Images/routericon2.JPG">  

Now we need to configure our newly created router to do this right click on the newly created router and click on the configure template option

<img src="Images/template.JPG">

From here go to the Slots tab and add "PA-GE" to Adapters slots 1 through 4 this will add 4 gigabyte interfaces to your routers when you spawn them

## Basic GNS3 Network

## Topology

The following network topology is what we will use to create our basic network in gns3

<img src="Images/topologycloud.JPG">

Lets create a simple network in GNS3 and connect it to our physical connect via a tap interface

In order to do this first we must create a new blank project and name it *SOMETHING*

The code to configure the gns3 network environment has been provided below<br>
The commands have also been provided in a way that they will work no matter where your are located cli wise *REWORD*

1) Add 2 of our newly created cisco 7200 routers
2) Cable these 2 routers together
3) Configure these 2 routers with ip addresses according to the topology provided

On R1
```
end
conf t
int g2/0
ip address 192.168.2.1 255.255.255.252
no shut
```
On R2

```
end
conf t
int g2/0
ip address 192.168.2.2 255.255.255.252
no shut
```
4) Verify that R1 can ping R2 and vice versa
On R1
```
end
ping 192.168.2.2
```
On R2
```
end
ping 192.168.2.1
```
5) Add a cloud to our network, you can find this by clicking on the end device icon<br>
<img src="Images/enddevice.JPG"><br>
<img src="Images/cloud.JPG">
6) Select the cloud
7) Go to the "TAP Interfaces" tab
8) Check the that tap1 has been added
<img src="Images/tap.JPG">
9) Cable R1 to the cloud using the tap1 interface
10) Configure R1 with an ip address

On R1 
```
end
conf t
int g1/0
ip address 192.168.1.1 255.255.255.0
no shut
```
11) confirm that R1 can ping the tap interface
```
end
ping 192.168.1.254
```
12) Confirm that the gns3server can ping R1
* you will need to do this on the gns3server

we now need to Configure OSPF and a static default route then redistirbute that route into ospf, this will allow R2 to send traffic to the gns3server and outwards *REWORD*

13) Configure OSPF

On R1
```
end
conf t
ip route 0.0.0.0 0.0.0.0 192.168.1.254
router ospf 1
router-id 1.1.1.1
network 192.168.1.0 0.0.0.255 area 0
network 192.168.2.0 0.0.0.3 area 0
default-information originate
```
On R2
```
end
conf t
router ospf 1
router-id 2.2.2.2
network 192.168.2.0 0.0.0.3 area 0
```
14) Confirm that R2 can ping tap1
```
end
ping 192.168.1.254
```
15) Confirm that the gns3server can ping R2
* you will need to do this on the gns3server


Our final step

Because the ansible requires the use of SSH to deploy playbooks, we will need to configure and enable SSH on our routers, a basic ssh configuration has been provided 

On both R1 and R2
``` 
end
conf t
ip domain-name ansible.com
crypto key generate rsa
1024
ip ssh version 2
username admin privilege 15 password 0 admin
line vty 0 4
login local
transport input ssh
exit
```

## Ansible Installation And Setup

## <p style="text-align: center;"> The following occurs on our gns3server VM

All that is left for us to do now is to get ansible setup and then we can run it against our gns3 topology

We now need to download and install ansible onto our server we can achieve this by using the following
```
sudo apt-get install ansible -y
```
Lets go to the ansible directory where the ansible.cfg and hosts file are stored, from this directory you can create and deploy your ansible playbooks as well as modify your host files

```
cd /etc/ansible/
```
We will disable host_key_checking in our ansible configuration file so that we don't need to ssh onto our gns3 routers first before we can deploy playbooks while this does save time it is a security risk, in order to do this we need to open our ansible configuration file in our text editor
```
sudo vim /etc/ansible/ansible.cfg
```
Go to line 62 and uncomment the following
```
#host_key_checking = False
```
Then save the file

Inside the hosts file you can define your network devices and asign them to groups an example is provided inside the file by ansible

<img src="Images/hosts.JPG">

In the hosts file you can define your environments in a few different ways you can have have them ungroup or you can put them into groups, having them in groups allows you to deploy your playbooks to a set of devices which can be helpful to make sure they are all configured the same.

In the /etc/ansible/hosts file we will add the ip addresses of the devices we wish to use ansible against
```
[network]
R2 ansible_host=192.168.1.1 ansible_network_os=ios ansible_ssh_user=admin ansible_ssh_pass=admin
R1 ansible_host=192.168.2.2 ansible_network_os=ios ansible_ssh_user=admin ansible_ssh_pass=admin
```
* The [network] defines the name of the group this can be called whatever you wish
* R2 and R1 are the names of the hosts
* ansible_host=X.X.X.X is the ip of the host
* ansible_network_os=ios defines the network platform that the host is using
* ansible_ssh_user=admin the user account that ansible uses to connect with in this example its admin because that is what we created earlier when we setup the router configuration in gns3
* ansible_ssh_pass=admin the password of the user account that ansible is using to connect with

<br>

Let's run an ad-hoc command against the hosts we just added

```
ansible all -c network_cli -m ping
ansible network -c network_cli -m ping
```

lets create a easy playbook to test if everything is working correctly
ansible can be a bit pedantic with its formating so here is a 

```
sudo vim /etc/ansible/ping.yaml
```

and copy and paste the following 
```
---
  - name: ping
    hosts: routers
    connection: local
    gather_facts: false
    tasks:
            - ping:
```

then write quit<br>

Now that we have created our playbook it is time to run in

in order to run your playbook you must be in the directory that it is located or specify the location
In order to run your ansible playbook that you have now created you need to be located in the directory that the playbook was made 

```
ansible-playbook ping.yaml
ansible-playbook /etc/ansible/ping.yaml
```

After running your playbook the following output should occur: <br>

<img src="Images/playbook2.JPG">
<br>

Congratulations you have now successfully deployed your first ansible playbook against a virtual network *REWORD*

In future labs we will cover more uses for ansible in both a local and cloud environment 

Please save your work or make a script to recreate it quickly along as future labs will be built off this

Further reading:

ansible module list can be found here 
* https://docs.ansible.com/ansible/latest/modules/modules_by_category.html

ansible playbooks user guide can be found here 
* https://docs.ansible.com/ansible/latest/user_guide/playbooks.html


<br>

# Lab 2 local + cloud versions

## Local

### Requirements

- VM Workstation 
- Ubuntu VM
- GNS3
- Windows Machine
- Completion of lab 1

## Basic Playbooks (pull configs etc?)

## pull configs i.e. backups?


```
sudo apt-get install tree
```
configure ip routes so that our playbooks can reach the devices
```
sudo ip route add 192.168.0.0/24 via 192.168.0.128 dev ens33
sudo ip route add 192.168.1.0/30 via 192.168.0.128 dev ens33
```

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

## run a check on your config backups to make sure that they are configured the same - the interface ip and such

## Cloud

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

download and install azure command line
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
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

# Lab 3 local + cloud versons

## Automate daily backup config playbook OR PHYSICAL?

## cron w/ ansible for backups

## some of the previous stuff but on physical gear?

## create a network with a playbook? (AZURE?)

