---
title: Configuring Traffic Mirroring
date: 2025-12-27
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - Traffic Mirroring
  - NSM
  - Security Onion
category: Set-up
---
## Getting Back to Work

It's good to get back after it after a few weeks away from this project due to the holidays and some travel. 

My immediate goal was to continue building out the infrastructure for this lab. First up, I had to find a way to mirror traffic so that I could run various NSM tools and get hands-on with detection rules, IDS, and more. This ended up being significantly more difficult than I had hoped.

I first attempted to simply mirror traffic to a second NIC on a Security Onion machine through a linux bridge built on the host. However, I had no luck here, and could not get traffic in the LAN of the homelab duplicated, even with the second NIC on the SO machine set to promiscuous mode. Next, I attempted to use the Open vSwitch tool, but ran into various errors when attempting to boot my VMs (I did not, but should have, logged these errors). 

After trying several methods and doing a lot of troubleshooting, I gave both VirtualBox and VMWare a go, but again, did not have any luck in getting traffic to mirror. Furthermore, trying to reconifigure almost everything I had already done was super frustrating.

I ended up coming back to KVM/QEMU and finding a resolution with a second attempt using Open vSwitch. First, I pinned each VM's NIC to a <target dev=' '> entry in its XML config to prevent the generation of random vnetX interfaces. Then, I ran `ip link | grep tap-` to ensure that each host had visibility on the pinned interfaces. I cleared out some old mirror objects leftover from previous troubleshooting with `sudo ovs-vsctl clear bridhe br0 mirrors` to start cleanly.

Next, I made a persistent mirror with:
```
sudo ovs-vsctl \
  -- --id=@dst get port tap-so-sniff \
  -- --id=@m create mirror name=so-mirror \
     select-all=true \
     output-port=@dst \
  -- set bridge br0 mirrors=@m
```
With that mirror created and all the VM's networks set to corresponding bridge, I was able to detect all packets traversing the network on the Security Onion server running in the LAN.

A quick `sudo tcpdump -i enp2s0` (monitoring NIC) on the server, followed up by some traffic generated on an Ubuntu desktop on the LAN, confirmed I could see all network traffic.
