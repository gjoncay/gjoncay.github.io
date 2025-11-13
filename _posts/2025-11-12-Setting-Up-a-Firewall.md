---
title: Setting up a pfSense Firewall VM
date: 2025-11-12
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - Firewall
  - pfSense
category: Set-up
---
## Downloading the iso File
I downloaded the iso from [pfsense.org](https://www.pfsense.org/download/). I had to create an account and add the (free) software to my account. 

The file was bundled in a .gz, so I `gunzip`'d it. 

## Creating the VM

Next, I created a new VM with the pfsense iso. Importantly, I attached it to both the existing network and added a NIC connected to a new isolated network.

![Isolated Network](assets/img/iso_network_creation.png)

![Two NICs](assets/img/double_NIC.png)

I then booted up the pfSense VM and began the installation process. I made sure the LAN NIC was set to static interface mode and the 192.168.200.0/24 network, with the other WAN NIC attached to the 192.168.100/24 network in DHCP interface mode.

After that, pfSense prompted me to install the community edition, which took a couple minutes.

![pfSense Installation](assets/img/pfsense_loading.png)

After the machine finished loading, I rebooted it to finish the installation process.

## Network Configuration

At the main menu, I selected 14 to enable ssh and then pinged `8.8.8.8` to verify connectivity.

I then shut down both the Wazuh server and the Ubuntu desktop machine I was using to monitor the dashboard and migrated them to the isolated network.

![Moving Networks](assets/img/network_move.png)

Now, although I set the network itself on the VM manager to 192.168.200.0/24, pfSense defaulted to 192.168.1.0/24. To fix this, I used the pfSense GUI accessed via my Ubuntu desktop VM to change the network address on the LAN interface to 192.168.200.1/24.

However, this created a bit of a snafu - as soon as I changed the address on the pfSense machine, I lost connection to the GUI (obviously). I had to jump over to the CLI on pfSense box and manually set the LAN interface to 192.168.200.1/24. I then ran the following on the desktop:

```bash
sudo ip addr add 192.168.200.10/24 dev enp1s0
sudo ip link set enp1s0 up
sudo ip route add default via 192.168.200.1
```

This allowed me to immediately recieve an address from the router (192.168.200.2), so I deleted the .10. However, despite being able to ping the router, I still could not reach the GUI.

I had to access the pfSense shell via option 8 from the console menu and run:

```
/etc/rc.restart_webgui
```

I then attempted to ping the pfSense box from the desktop and vice versa, with success on both ends.

With fingers crossed, I again attempted to log into the pfSense GUI at http://192.168.200.1.

![GUI Login](assets/img/GUI_login.png)

With that, I'm going to wrap up this post. I didn't end up documenting all the trouble shooting that went in to reconfiguring the networks and ensuring I could access the GUI from the LAN because it was mostly me making mistakes/typos/etc... but what's new?

I'll continue this series with further configuration on the pfSense machine and adjustments to the network architecture to better emulate a real environment.