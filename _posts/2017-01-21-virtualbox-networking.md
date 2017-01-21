---
layout: post
title: VirtualBox networking with Ubuntu
modified:
excerpt: "How to configure VirtualBox to run multiple guest VMs on the same network"
comments: true
tags: [tech,virtualbox,networking,nat,host-only]
image:
  feature: network.jpg
  credit: Quinn Dombrowski
  creditlink: https://www.flickr.com/photos/quinnanya/
---

This post aims to explain how to set-up a working network configuration for a group of VirtualBox VMs running on a single host machine.

I needed to be able to get the guest VMs communicating with one another so that I could set up and test a clustering solution for a project I was working on.  I wanted to be able to individually SSH into each guest VMs from the host laptop.  I didn't need the guests to be visible outside of my laptop.

This post assumes that you know your way around Ubuntu server and also that you've already created a VM within VirtualBox.


### Operating system and version information

This is what I was running:-

- Laptop Windows 10 Professional Anniversary edition
- VirtualBox 5.0.30
- Guest VM(s) running Ubuntu 14.04



### Overall VirtualBox network settings

You only need to do this once.

1. In VirtualBox, go to "*File*" - "*Preferences*" - "*Network*"
1. Add a "*Host-only Network*"
1. Select it and click the screwdriver button to edit the details
1. Leave the "*Adapter*" tab as is.  It should say:-
  - "*IPv4 Address*" = "*192.168.56.1 *"
  - "*IPv4 Network Mask*" = "*255.255.255.0*"
1. On the "*DHCP Server*" tab, ensure "*Enable Server*" is selected
1. Make these settings:-
  - "*Server Address*" = "*192.168.56.1*"
  - "*Server Mask*" = "*255.255.255.0*"
  - "*Lower Address Bound*" = "*10.13.13.101*"
  - "*Upper Address Bound*" = "*10.13.13.254*"
  
This will create a new adapter within Windows.  You can see it (if you really want to) by clicking:-

1. "*Settings*" (i.e. the gear icon) on the **Windows** home menu
1. "*Network & Internet*"
1. "*Change adapter options*" (you shouldn't change anything, you'll just be able to see it in the list)


### Individual server settings  

Do this for each of your guest VMs.

- In VirtualBox, select your server and then go to "*Settings - Network*"

#### Adapter 1

1. Select the "*Adapter 1*" tab
1. Ensure "*Enable Network Adapter*" is selected
1. Set "*Attached to*" = "*NAT*"
1. Expand the "*Advanced*" section
  - Set "*Adapter Type*" = "*Intel PRO/1000 MT Desktop*"
  - Click the "refresh button" (circular arrows) next to the MAC Address to generate a fresh one
  - Ensure "*Cable Connected*" is selected

##### Port forwarding

- Add a rule, called "Rule 1" or whatever you like
  - "*Protocol*" = "*TCP*"
  - Choose a IP address in the range `127.0.1.[1-255]`.  This will be the address you will use to SSH into the box from the host machine - i.e. your laptop.  Type this into the "*Host IP*" column.  Ensure that you choose a different address for each VM
  - Set "*Host port*" = "*22*"
  - Leave "*Guest IP*" empty.  What will happen is that VirtualBox will assign IP address `10.0.2.15` to the VM's `eth0` interface.  All VMs will be given this same IP address but it doesn't matter as it's not used for inter-communication
  - Set "*Guest Port*" = "*22*"

You can also add any other rules you like in order to give your host access to services on the guest - e.g. a rule for port 80 if you're running a web server and want to browse from your host's desktop.


#### Adapter 2

1. Select the "*Adapter 2*" tab
1. Set "*Attached to*" = "*Host-only Adapter*"
1. Choose "*Name*" fom the drop-down box to be whatever name it was given when you set up the Overall VirtualBox Network settings above (you'll probably only have one option)
1. Expand the "*Advanced*" section
  - Set "*Adapter Type*" = "*Intel PRO/1000 MT Desktop*"
  - Set "*Promiscuous Mode*" = "*Allow All*"
  - Click the "refresh button" (circular arrows) next to the MAC Address to generate a fresh one
  - Ensure "*Cable Connected*" is selected


### SSH access

You will be able to SSH into each guest from your desktop using the IP address you chose in the Adapter 1 port forwarding section (e.g. `127.0.1.2`).


### Ubuntu configuration

You'll need to ensure that each Ubuntu VM knows that it's been given two network interfaces.

SSH into each guest and do this:-

    `sudo vi /etc/network/interfaces'

Make sure it looks something like this:-

    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).
    
    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface to allow guest VM access from the host and for outgoing comms
    auto eth0
    iface eth0 inet dhcp
    
    # Secondary network interface for inter-VM comms
    auto eth1
    iface eth1 inet dhcp

Whilst you're on the server, update the VM's hostname and IP addresses in `/etc/hosts`.  It should look something like this (of course, your IP address may vary):-

    127.0.0.1       localhost
    127.0.1.1       yourserver
    10.13.13.102    localhost
    10.13.13.102    yourserver

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters

Also put the VM's hostname into `/etc/hostname`.

Obviously, you can do normal Ubuntu things to set up static IP addresses, etc.  .  There may be value in doing this to *eth1*, but there's not much point doing it to *eth0*, as it's all alone on its own virtual network.


### Inter-VM communication

The VMs will now be able to talk to each other via the IP addresses assigned to 'eth1' (e.g. `10.13.13.102`).


### Creating a new guest VM

Quick instructions on cloning an Ubuntu VM and configuring it:-

1. In VirtualBox, right-click the VM you want to clone and select the "*Clone...*" option
1. Make sure you click the "*Reinitialise MAC address of all network cards*" option
1. Once cloned, go into the "*Settings*" of the newly created VM and choose "*Network*", "*Adapter 1*", "*Advanced*" and then "*Port Forwarding*" (I'm assuming the VM has been configured as above, meaning that this is the NAT adapter)
1. Pick an unused IP address for the "*Host IP*" (i.e. add 1 or more to the last octet - making sure it's unique across your VMs)
1. Launch the server and SSH into it
1. If you want your servers to have static IP addresses, do the necessary updates
1. `sudo vi /etc/hostname` and give it a new name
1. `sudo vi /etc/hosts` and make sure there are entries for both localhost and the new hostname against both of the IP address chosen above (i.e. the one you're using for ssh access) and the one assigned to eth1