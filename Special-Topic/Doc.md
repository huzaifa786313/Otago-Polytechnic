# IN730 Special Topic - Network Automation
<br>

## Initial Setup
In the following lab we will install and configure the required components needed to **SOMETHING** gns3 and ansible **EXPAND**

### Local

#### Topology

<img src="Images/topology.JPG">

#### Requirements

- VM Workstation 
- Ubuntu VM
- GNS3
- Windows Machine

#### Setup

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
exit
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

This means that ansible can successfully 
A comprehensive list of the modules that are avaliable can be found here https://docs.ansible.com/ansible/latest/modules/modules_by_category.html



ansible all -c network_cli -u samsojl1 -k -m ping -e ansible_network_os=ios

## Troubleshooting

If at some point your pings / connection stops working between your linux vm and your 
delete the cable connecting R1 and the cloud together then reconnect

# Azure

## Disclaimer

vim is the text editor used in the following lab guide but you can use your own preferred text editor if you wish *SOMETHING ABOUT HAVING TO KNOW IT YOURSELF*

Ansible / gns3server are the same *REWORD*

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

we now need to create a network that will be used *REWORD*

go to your ansible resource group and click the add button

go to the networking tab and select virtual network

Name - ansible
Region - Australia East

ipv4 address space
192.168.0.0/16

add subnet
name - ansible
subnet 192.168.0.0/24

review + create
create

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

Before we start lets make sure that our software is up to date *REWORD*

```
sudo apt-get update -y
```
<br>
First thing we need to do is install gns3 server onto our linux server so that our gns3 client can connect to

run the following commands to install gns3 server
```
cd /tmp
curl https://raw.githubusercontent.com/GNS3/gns3-server/master/scripts/remote-install.sh > gns3-remote-install.sh
sudo bash gns3-remote-install.sh --with-openvpn --with-iou --with-i386-repository
```

Its now time to edit our server settings so that we can connect to it using the gns3client virtual machine in order to do this we need to edit the gns3_server.conf file

```
sudo vim /etc/gns3/gns3_server.conf
```

change the host = variable to the ip of your machine<br>
( you can check this by running the following command )
```
ip a
```
and the port = variable to 3081

Your gns3_server.conf file should look like the one in the image below

<img src="Images/serverconf.JPG">

After making these changes restart your gns3 using the following command
```
sudo systemctl restart gns3.service
```

now we need to create a TAP interface that the gns3 client can use to connect to the gns3 server so that it can communicate with outside devices

First we will need to download uml-utilities which will allow us to create TAP interfaces
```
sudo apt-get install uml-utilities
```

Now that we have uml-utilties we can go ahead and create a TAP interface
* Do note that the TAP interface and the ip associated with it are not persistant doing it the following way
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

Create routes
```
sudo ip route add 192.168.1.0/24 via 192.168.1.254 dev tap1
sudo ip route add 192.168.2.0/30 via 192.168.1.254 dev tap1
```

## Install GNS3 Client

Step X) GNS3 Setup

X) Sign up to gns3 https://www.gns3.com/ then proceed to download the windows version of gns3 https://www.gns3.com/software/download

GNS3 INSTALL STEPS HERE

Edit > preferences > server

change the host to gns3server.australiaeast.cloudapp.azure.com<br>
port 3081

<img src="Images/server.JPG">

leave auth unchecked

Step X) Router Template Configuration

## * *change to a more permament storage - TBD* *
X) Download the image for the cisco c7200 router here https://otagopoly-my.sharepoint.com/:f:/g/personal/samsojl1_student_op_ac_nz/EvYyb9R7e6FMkohq9r1w4rgBG-3bAONCIjTJKHz6J0xwdg?e=p0iyG3

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
X) Verify that R1 can ping R2 and vice versa

X) lets add a cloud to connect our virtual routers to our physical network 

After adding the cloud click on it and go to "Ethernet Interfaces" tab, then tick the "Show special Ethernet interfaces" box, click the Add all button, this will add all the interfaces to the cloud allowing you to connect your virtual router to it

now lets connect a cable between our cloud and R1

we will connect to the tap1 interface on the cloud and g2/0 on R1

now we will apply an ip address to g2/0 in the same subnet as tap1 allowing connection

On R1 
```
end
conf t
int g1/0
ip address 192.168.1.1 255.255.255.0
no shut
```

we now need to Configure OSPF and a static default route then redistirbute that route into ospf, this will allow R2 to send traffic to the gns3server and outwards *REWORD*

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

You should now be able to ping from your gns3server to R2

Our final step

Because the ansible program uses SSH to deploy playbooks as well as ad-hoc commands to devices, we will need to enable SSH on our routers, a basic configuration has been provided 

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

## Step X) Ansible Installation And Setup

All that is left for us to do now is to get ansible setup and then we can run it against our gns3 topology


We now need to download and install ansible onto our server we can achieve this by using the following


```
sudo apt-get install ansible -y
```


X) lets go to the ansible directory where the ansible.cfg and hosts file are stored, from this directory you can create and deploy your ansible playbooks as well as modify your host files

```
cd /etc/ansible/
```

We need to disable host_key_checking so that we aren't forced to ssh onto our gns3 routers first in order to do this we need to 

```
sudo vim /etc/ansible/ansible.cfg
```

go to line 62 and uncomment the following

```
host_key_checking = False
```

then save the file


Inside the hosts file you can define your network devices and asign them to groups an example is provided inside the file by ansible

in the hosts file you can define your environments in a few different ways you can have have them ungroup or you can put them into groups, having them in groups allows you to deploy your playbooks to a set of devices which can be helpful to make sure they are all configured the same.

in the /etc/ansible/hosts file we will add the ip addresses of the devices we wish to use ansible against

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

let's run an ad-hoc command against the hosts we just added

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

then write quit
<br>

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


A comprehensive list of the modules that are avaliable can be found here https://docs.ansible.com/ansible/latest/modules/modules_by_category.html





# FIX THIS 
Windows VM

go to https://www.gns3.com/ and sign up 

download gns3 windows version

download c7200 router image to be used with gns3 here
https://otagopoly-my.sharepoint.com/:f:/g/personal/samsojl1_student_op_ac_nz/EvYyb9R7e6FMkohq9r1w4rgBG-3bAONCIjTJKHz6J0xwdg?e=fI4JFa

Linux VM



#### Req


<br>

# Lab 2

## Basic Playbooks (pull configs etc?)

<br>

# Lab 3

## Automate daily backup config playbook OR PHYSICAL?
