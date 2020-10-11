# IN730 Special Topic - Network Automation
<br>

## Lab2 - Ansible Playbooks (Cloud Version)

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

