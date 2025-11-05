---
title: Starting Up an Alpine Linux VM
date: 2025-11-05
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
category: Set-up
---


First, I defined the network with the following xml data:

```
cat <<'EOF' > 05NOV_Network.xml
<network>
  <name>mini-lab-net</name>
  <forward mode='none'/>
  <ip address='192.168.100.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.100.10' end='192.168.100.250'/>
    </dhcp>
  </ip>
</network>
EOF
```

Next, I ran the following:
```
sudo virsh net-define 05NOV_Network.xml
sudo virsh net-start 05NOV_network
```

Stopping the network was just as easy:
```
sudo virsh net-destroy 05NOV_Network
```


I ran into some initial issues with permissions and ensuring the VM management stack could function properly. After some StackOverflow, Google, and ChatGPT trouble shooting, I ran the following:

```
sudo mkdir -p /var/lib/libvirt/boot
sudo cp ~/Downloads/alpine.iso /var/lib/libvirt/boot/
sudo chown root:kvm /var/lib/libvirt/boot/alpine.iso
sudo chmod 644 /var/lib/libvirt/boot/alpine.iso
```

This allowed the following command to work:
```
sudo virt-install \
  --name alpine-base \
  --ram 512 \
  --vcpus 1 \
  --disk path=/var/lib/libvirt/images/alpine-base.qcow2,format=qcow2 \
  --cdrom /var/lib/libvirt/boot/alpine.iso \
  --network network=05NOV_network \
  --graphics none \
  --console pty,target_type=serial \
  --noautoconsole \
  --osinfo linux2024
```
I was subsequently able to connect to the Alpine box with:
```
sudo virsh console alpine-base
```


After logging in with `root` at the login prompt, I could proceed through the setup wizard, mostly choosing defaults. Of note, this box has no connectivity, and thus is pretty minimalistic at the moment.

And there we go! I was worried about how difficult the set-up would be after coming from a more user-friendly GUI like VMware, but it was not terrible. Time to move on to expanding the network and getting some services up and running.
![Virtual Machine Manager with Alpine](assets/img/alpine_running.png)





