---
title: Verifying Connection Between Servers
date: 2025-11-09
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - SSH
category: Set-up
---
Baby steps! I'm just looking to SSH between two new Ubuntu servers that I had just set up with a KVM-QEMU stack. Both are running Ubuntu 24.04.3 LTS.

(Side note... I need to decide on a tense for these posts... we'll do present tense I suppose.)

First, I am grabbing the v4 IP on each box with: `ip addr show`.

Ubuntu1 is rocking .232 on the private /24 network I set up for this lab. Ubuntu2 is on .176.

Both are able to ping each other as well as external addresses, as they were set up in a NAT config on initial installation. I looks like DNS is working without issue as well.

Given that I installed OpenSSH during the initial Ubuntu installation, we should be good after ensuring ssh is actually running.

On ubuntu2, I can check the status of the ssh daemon with: `sudo systemctl status ssh`, which yielded the following:
![SSH Status](assets/img/ssh_status.png)

I can then start it and enable it and ensure it starts on boot with:
```
sudo systemctl start ssh
sudo systemctl enable ssh
```
Then, after verifying I'm not going to be blocked by ufw, I can connect with: `ssh ubuntu2user@192.168.100.176`

Connected!

