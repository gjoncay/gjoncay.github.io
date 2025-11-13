---
title: Setting up a pfSense Firewall VM (Pt. 2)
date: 2025-11-13
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - Firewall
  - pfSense
category: Set-up
---
## Continuing pfSense Configuration

Today, I wanted to make some progress on the network topology to better reflect a real environment, which included ironing out some wrinkles in the pfSense machine. First off, I adjusted the DHCP pool the router/firewall was providing to the LAN network to 192.168.200.2-50/24. I then added static IPs for both the ubuntu desktop and the Wazuh server.

To do so, I navigated to the DHCP menu under services and then scrolled to the bottom of the page. After clicking 'Add Static Mapping', I entered the MAC address and hostname of the Wazuh server. I then toggled the network interface on the Wazuh server with:

```bash
sudo ip link set enp1s0 down
sudo ip link set enp1s0 up
```

Checking the address on the interface with `ip a` yielded an IPv4 of 192.168.200.51.

I repeated this process with the desktop, setting it to 192.168.200.51.

After configuring IPs, I pinged each machine from the other to verify again that everything within the LAN was talking and that the default route was good to go with `ip route show`.

However, I again ran into issues accessing the pfSense web GUI from the desktop. Although I could ping the pfSense machine from the desktop, I couldn't connect to the GUI and could not ping the desktop from the pfSense console.

Rebooting the pfSense machine and then restarting the GUI with:

```
/etc/rc.restart_webgui
```

resolved the issue. 

## Adding Another LAN

My next goal was to create another LAN aside from the existing network with the Wazuh server and desktop (serving as a management LAN), and then place the already provisioned Ubuntu servers into it. I'd like them to each run Wazuh agents that can talk back to the server in the management network.

I snapshotted everything, and then created another network in the KVM/QEMU manager. I then added a new interface to the pfSense machine (after shutting in down) in the VM manager and booted it back up.

I moved to the interface set up menu, and registered the new NIC.

![Adding a Network](assets/img/pfsense_13NOV_1.png)

I then had to set interface IP addresses for that network. I selected option 2 on the console menu, and entered 192.168.151, 24 for the subnet bit count, and chose not to set up IPv6, and enabled DHCP with a range of 192.168.150.2-50. 

![Adding a Network](assets/img/pfsense_13NOV_2.png)

I then changed the interfaces on each of the ubuntu servers I created a couple days ago to reside on the 192.168.150.0/24 network.

![Adding a Network](assets/img/pfsense_13NOV_3.png)

Both successfully recieved IP addresses in the correct network from the pfSense machine.

![Checking IPs](assets/img/pfsense_13NOV_4.png)

I then ran `sudo systemctl status wazuh-agent.service` and saw it was running and enabled on both ubuntu servers.

However, I had not yet added rules to allow that traffic to traverse networks to the SIEM.

I added a pass rule on the 192.168.150.0/24 interface to allow traffic from that network to the address of the SIEM server on ports 1514 and 1515 (per Wazuh documentation).

Finally, I edited the config file with:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

and replaced the old server IP with the correct address.

After restarting the service with:

```bash
sudo systemctl restart wazuh-agent.service
```

I refreshed the Wazuh dashboard, and saw I once again had connectivity between the SIEM server and the Ubuntu server running the agent, but between networks.

![Agents Connected](assets/img/pfsense_13NOV_5.png)

I repeated this process on the other server, and was good to go (at least for this post)!



