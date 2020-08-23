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

D) Edit > virtual network editior > change settings > make sure that you are bridging to the physical interface

Step X) GNS3 Setup

A) Sign up to gns3 https://www.gns3.com/ then proceed to download the windows version of gns3 https://www.gns3.com/software/download



Router Template Configuration

A) Download the image for the cisco c7200 router here https://www.dropbox.com/sh/hhzpveww67m8ifl/AAClzT4W0LkndrIMcxA6ubx3a/GNS3/Lab%20D%20drive%20with%20VIRL%20V225/GNS3/images/IOS?dl=0&subfolder_nav_tracking=1 

B) Import the C7200 Router into gns3 by going file > new template > install an appliance from the GNS3 server > then click the dropdown for the routers section and select Cisco 7200 then click install > Install the appliance on your local computer > create a new version, call it whatever you wish > select your version from the list and click import, locate and select the c7200 bin file your downloaded earlier > next > accept the install > finish, if you click on the router icon on the left hand side you should now see your router template you installed



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

exit ```

#### Azure

##### Req



# Lab 2

## Basic Playbooks (pull configs etc?)

# Lab 3

## Automate daily backup config playbook
