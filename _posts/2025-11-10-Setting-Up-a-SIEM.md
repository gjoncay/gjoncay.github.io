---
title: Setting Up a SIEM (Pt. 1)
date: 2025-11-10
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - SIEM
  - Wazuh
category: Set-up
---
## Getting Started
To start, I set up another Ubuntu server running 24.04.3 LTS with 8GB RAM, 100GB storage, and 2 CPUs. I could probably get away with lower specs, but I'd like to eventually expand this lab quite a bit, and thus have configured the server running Wazuh to be (hopefully) sufficient for monitoring 10+ machines and a good amount of traffic.

Once the VM was up and running, I verified connectivity and ensure it's settled into the same network as the existing two Ubuntu machines.

Now it's was time to get after installing Wazuh, an open source SIEM and EDR tool that will be perfect for catching logs, finding vulns, and acting as a focal point for most of my other goals within this lab.

I used Wazuh's official documentation, which can be found [here](https://documentation.wazuh.com/current/quickstart.html).

First, I made sure the server is up to date:
```bash
sudo apt update && sudo apt upgrade -y
```
Then, I ran:
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
The installation took about 5 minutes, during which I started to download another iso file to run Ubuntu desktop. I'll access the Wazuh dash from this desktop environment, which will run from another system in the lab network. There are a couple different ways to access the dashboard depending on lab set up, but this seems to be the most straightforward.

I then ran the following to make sure the three services associated with Wazuh are running correctly:
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
All three commands indicated the services were enabled and actively running.

## Checking the Dashboard
Once I downloaded and installed Ubuntu desktop on a new VM (with 8GB RAM and 80GB storage), I had to check the port for the Wazuh dashboard on the host server to ensure the listener was up. To do so, I ran:
```bash
sudo ss -tulnp | grep wazuh
```
As an aside, those switches show tcp and udp listening sockets without resolving names and showing process info.

That displayed listeners on both 1514 and 1515, but for "wazuh-remoted" and "wazuh-authd", which were not the dashboard itself.

Instead, I ended up finding the correct port with:
```bash
sudo ss -tulnp | grep node
```
Which returned "node" listening on port 443.

On the desktop VM, I could now connect to the dashboard at the private IPv4 of the Ubuntu server.
![Wazuh Login](assets/img/wazuh_login.png)

Though I read online that creds would be admin/admin... that was not the case. I am not sure if something has changed with Wazuh initial config and installation, but I was able to eventually login thanks to [this reddit post](https://www.reddit.com/r/Wazuh/comments/utfk8l/whats_the_default_login_id_password_for_the/). In short, I had to extract the wazug tarball then cat wazuh-passwords.txt, which was located inside and provided a randomly generated password for the dashboard (as well as various other users for tasks within Wazuh).

Once logged in, I was able to see the dash and check out the GUI for the first time! Overall set up was not bad at all, and I'm looking forward to installing other agents on endpoints in the lab.

![Wazuh Dashboard](assets/img/wazuh_dash.png)