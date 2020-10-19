# IN730 Special Topic - Network Automation

## Lab3 

## Ansible on physical gear

## Requirements

- VM Workstation 
- Windows Machine
- Completion of lab 1 and 2
- 2 Routers
- 1 Switch

This section will combine aspects from Lab 1 and 2 and apply them towards physical equipment

First thing we need to do is download a ubuntu image that we will use to create our linux VM that will be used for as the ansible server

- Download Ubuntu 20.04.1 LTS image from here https://ubuntu.com/download/desktop

After downloading the ubuntu image we will now create a virtual machine in VM Workstation

- Launch the VM Workstation Application 
- Go to File
- Select New Virtual Machine 
- Select the "Typical (recommended)" option 

<img src="Images/typical.PNG">

- Select "Installer disc image file (iso):" then locate the Ubuntu iso file you downloaded earlier

<img src="Images/isoimage.JPG">

- Personalize your ubuntu machine how you wish

- Leave the "Specify Disk Capacity" with the defaults 

<img src="Images/disksize.JPG">

- Customize Hardware and change the RAM to 4GB

<img src="Images/hardware.JPG">

- Finish


After creating the linux VM we now need to configure some network options in VM workstation

- Click on the Edit tab and go click on the virtual network editior 

- Click on the "Change Settings" option and accept the administrator promopt 

<img src="Images/admin.JPG">

- Select VMnet0 then select Bridged and bridge it to your machines physical network interface

<img src="Images/bridged.JPG">

- Also select the VMnet that has the type and external connection of NAT and change its subnet ip to 192.168.0.0 with a subnet mask of 255.255.255.0

<img src="Images/subnetd.JPG">

In the end your Virtual Network Editor should look simillar to the image below

<img src="Images/virtualnetworkeditor.JPG">


- Connect to your linux VM and open a terminal

- use the command "ip a" and note down the ip address on the ens33(ens number may vary but there will be only one)

<img src="Images/ens.PNG">

This ip will be used later

## Create a simple network

Lets create a simple network

<img src="Images/topologyphysical.PNG">

- Cable these 2 routers together according to the topology above
- Console onto R1/R2


- Cable R1 and the PC into the switch according to the topology

We need to set hostnames on R1 and R2 because when we create the crypto keys later on they require that the hostname be swaped from the default

On R1
```
en
conf t
hostname R1
```

On R2
```
en
conf t
hostname R2
```

Now configure the interfaces between R1 and R2

On R1
```
end
conf t
int g0/1
ip address 192.168.1.1 255.255.255.252
no shut
```
On R2
```
end
conf t
int g0/1
ip address 192.168.1.2 255.255.255.252
no shut
```
Verify that R1 can ping R2 and R2 can ping R1

Now configure the interface that is connected to our windows machine

On R1 
```
end
conf t
int g0/2
ip address 192.168.0.1 255.255.255.0
no shut
```

Configure a static default route and OSPF  then redistirbute that route into ospf

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
network 192.168.1.0 network 0.0.0.3 area 0
```

Because Ansible is agentless and uses SSH to deploy playbooks, you will need to configure and enable SSH onto your GNS3 Routers, a basic configuration has been provided 

``` 
end
conf t
ip domain-name ansible.com
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
username admin privilege 15 password 0 admin
line vty 0 4
login local
transport input ssh
exit
```

On your linux vm you will need to configure a route so that traffic knows where to go to to get to your routers

In your linux terminal use the following commands

```
sudo ip route add 192.168.0.0/24 via 192.168.0.128 dev ens33
sudo ip route add 192.168.1.0/30 via 192.168.0.128 dev ens33
```

Do note that routes configured this way aren't persistent and will need to be re entered, you can configure them to be persistent but for what we are trying to do that isn't required

Before switching from Live to Test port on our machine and therefor lose internet connection lets first download and install some software that we will need

```
sudo apt-get update
sudo apt-get install -y ansible
sudo apt-get install -y tree
sudo apt-get install -y net-tools
sudo apt-get install -y vim
```
Now to change the ip address of the interface that we are using on our windows machine

We can do this by doing the following

- Open the Control Panel
- Network and Sharing Center
- Change adapter settings
- Right click Ethernet 6 -> properties

<img src="Images/ipadapters.PNG">

- Accept

<img src="Images/ipprops.PNG">

- Internet Protocol Version 4 -> properties
- Select "Use the following IP address"

Input the following ip address 

- IP Address: 192.168.0.2
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.0.1

<img src="Images/ipsettings.PNG">

- Click "Ok" to confirm the settings
- Click "Ok" to exit

Now we can change from the L port to the T port so that your machine is now plugged into R1

Normally when working in the network room we would launch a windows VM that we would then turn the firewall off to allow pings and connection from the routers to the machine

In order for our routers to be able to ping our windows device we need to create a firewall rule

Here is a guide on how to Add IP Address in Windows Firewall

- On the Start menu, Click ‘Windows Firewall with Advanced Security’.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-1-1024x700.png">

- Click the ‘Advanced settings’ option in the sidebar.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-2-1024x700.png"> 

- On the left side, click the option ‘Inbound Rules’.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-3-1024x701.png"> 

- On the right, under the section ‘Actions’, click on the option ‘New Rule’. Windows Firewall shows you the New Inbound Rule Wizard.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-4.png"> 

- A new window will open and Select the ‘custom’ option and click Next.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-5.png"> 

- In the left-hand side again, go to the option ‘Scope’.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-6.png"> 

- Add the IP address and click on the ‘Ok’ button.

<img src="Images/How-to-Add-IP-Address-in-Windows-Firewall-7.png"> 

- add 192.168.1.0/30
- add 192.168.0.0/24

That is how you add an IP address to the windows firewall.



## Automate Ansible Playbooks

We will expand upon our backup script by automating it so that it will backup our router configs daily so that if we need to revert we can easily

Lets go into cron
```
crontab -e
```
And insert the following at the bottom
```
1 0 1-31 * * ansible-playbook /etc/ansible/backup.yaml
```

Your crontab should look simillar to the image below

<img src="Images/crontab.PNG">

This will run our playbook everyday at 00:01

You can use this same principal in order to run other playbooks that you may wish to automate
