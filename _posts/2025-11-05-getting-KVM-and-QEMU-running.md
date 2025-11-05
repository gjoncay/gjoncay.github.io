---
title: Getting KVM and QEMU Running
date: 2025-11-04
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
category: Set-up
---

## **How chunky is my machine?**

For some background, I am on a Thinkpad P16v Gen 2... luckily we have plenty of RAM for some VMs! Without knowing any better, the internet is telling me that KVM + QEMU is the way to go. I've used both VirtualBox and VMware in the past, but KVM's close to bare-metal performance is appealing. Hell, maybe I'll go crazy with the number of boxes I'm running in the lab.

![Fastfetch system stats](assets/img/system_stats.png)

## Installing KVM, QEMU, and Virt-Manager
First off, I'm verifying my CPU to ensure we are good to go with virtualizing some machines. Good to get some extra practice with egrep, even if this is as simple an example as it gets! This is also a good reminder to take a 100th look at regex formatting with the hope that someday it'll become easier.

```bash egrep -c '(vmx|svm)' /proc/cpuinfo```

The result was '44'. Because I was curious, I looked into what that actually means, and found that I have 22 CPU cores with 44 total threads. The command `nproc` yields 22 CPUs supporting 44 threads. Basic info for some, but others have plenty to learn :).

Next, I checked to ensure the kernel KVM modules are loaded with 
`lsmod | grep kvm`
, yielding:

```
kvm_intel             577536  0
kvm                  1466368  1 kvm_intel
irqbypass              16384  1 kvm
```
Now it's time to actually install some tools. I'm grabbing:
- KVM: virtualization
- QEMU: emulator/virtualization backend
- libvert: management
- bridge-utils: virtual networking
- virt-manager: GUI to keep it easy

With:
```
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

I'll also need to ensure I can run VMs without having to invoke `sudo` every time with: 

`sudo usermod -aG libvirt,kvm $USER`.

With `sudo systemctl status libvirtd`, I get:
```
● libvirtd.service - libvirt legacy monolithic daemon
     Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-11-04 15:29:08 HST; 6s ago
 Invocation: 8ae823df23ca46839284628b50888723
TriggeredBy: ● libvirtd.socket
             ● libvirtd-admin.socket
             ● libvirtd-ro.socket
       Docs: man:libvirtd(8)
             https://libvirt.org/
   Main PID: 224570 (libvirtd)
      Tasks: 21 (limit: 32768)
     Memory: 8.1M (peak: 9.9M)
        CPU: 1.633s
     CGroup: /system.slice/libvirtd.service
             └─224570 /usr/sbin/libvirtd --timeout 120

Nov 04 15:29:08 GrantsThinkpad systemd[1]: Starting libvirtd.service - libvirt legacy monolithic daemon...
Nov 04 15:29:08 GrantsThinkpad systemd[1]: Started libvirtd.service - libvirt legacy monolithic daemon.
```
Finally, with `sudo virsh list --all`, I get an empty list, meaning everything should be good to go (though no VMs are running yet).
```
 Id   Name   State
--------------------
```
## Launching the GUI
When trying to launch the GUI with `virt-manager &`, I got the following error:

![Virt error](assets/img/virt-manager-error.png)

It turns out I had to create the proper system user with: `sudo adduser --system --group --no-create-home libvirt-qemu`, then add them to the correct group with: `sudo usermod -aG libvirt,kvm $USER; 
newgrp libvirt`. Finally, I was able to restart libvirt:
```
sudo systemctl daemon-reload
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
sudo systemctl status libvirtd
```

Perfect! Running `virt-manager &`  doesn't throw any errors, and we should be ready to get some VMs running.

One final note: pinning the VM Manager to the dock continued to throw a connection error. This was a good opporuntity to learn about the shell environment. Logging in and out fixed the issue!






