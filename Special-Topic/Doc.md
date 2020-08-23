# IN730 Special Topic

## Lab 1

### Initial Setup

#### Local

##### Req

- VM Workstation 
- Ubuntu VM
- GNS3
- Windows Machine

##### Setup

Step X) Ubuntu Setup

A) Download Ubuntu image from here https://ubuntu.com/download/desktop

B) Launch the VM Workstation Application

C) File > New Virtual Machine > Typical > select browse from the installer disc image file(iso) option, then locate the iso file you downloaded > next > follow the onscreen instructions > finish

D) Edit > virtual network editior > change settings > make sure that you are bridging to the physical interface, note the subnet address on the VMnet interface

E) connect to your linux VM and open a terminal, then run the command "ip a" and note down the ip address on the ens33(ens number may vary but there will be only one) interface

Step X) GNS3 Setup

A) Sign up to gns3 https://www.gns3.com/ then proceed to download the windows version of gns3 https://www.gns3.com/software/download



Step X) Router Template Configuration

A) Download the image for the cisco c7200 router here https://www.dropbox.com/sh/hhzpveww67m8ifl/AAClzT4W0LkndrIMcxA6ubx3a/GNS3/Lab%20D%20drive%20with%20VIRL%20V225/GNS3/images/IOS?dl=0&subfolder_nav_tracking=1 

B) Import the C7200 Router into gns3 by going file > new template > install an appliance from the GNS3 server > then click the dropdown for the routers section and select Cisco 7200 then click install > Install the appliance on your local computer > create a new version, call it whatever you wish > select your version from the list and click import, locate and select the c7200 bin file your downloaded earlier > next > accept the install > finish, if you click on the router icon on the left hand side you should now see your router template you installed

Step X) Basic Network

Lets create a simple network in GNS3

A) create a new blank project and call it whatever you want

B) lets add 2 of our newly created routers to the project by going to the router tab and dragging 2 on, cable these 2 routers together and configure them using a private ip 

On R1
```
conf t
int g1/0
ip address 192.168.1.1 255.255.255.252
no shut
```
On R2
```
conf t
int g1/0
ip address 192.168.1.2 255.255.255.252
no shut
```
C) Verify that R1 can ping R2 and vice versa

C) lets add a cloud to connect our virtual routers to our physical network 

After adding the cloud click on it and go to "Ethernet Interfaces" tab, then tick the "Show special Ethernet interfaces" box, click the Add all button, this will add all the interfaces from your physical machine to the cloud allowing you to connect your virtual router to it



Step X) Ansible Setup

A) On your Linux VM open a terminal

B) run the command sudo apt-get install ansible and accept, this will install ansible onto your linux machine

C) cd into /etc/ansible/ this is where the ansible.cfg and hosts file exist from here you can create your playbooks

Inside the hosts file you can define your network devices and asign them to groups an example is provided inside the file by ansible

You can run either use ad-hoc commands or ansible playbooks 

Create a default route and redistribute it into ospf

disable host_key_checking
on line 62 uncomment host_key_checking = False

You should now be able to ping from your linux vm to R2

Because ansible uses SSH to deploy ad-hoc commands as well as playbooks you will need to enable SSH onto your GNS3 Routers, a basic configuration has been provided to copy and paste

``` conf t
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

#### Azure

##### Req



# Lab 2

## Basic Playbooks (pull configs etc?)

# Lab 3

## Automate daily backup config playbook
