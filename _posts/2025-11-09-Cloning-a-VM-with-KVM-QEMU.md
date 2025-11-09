---
title: Cloning an Alpine VM running on KVM QEMU
date: 2025-11-09
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
category: Set-up
---

The goal for the quick post is to continue to build familiarity with command line management of VMs as well as build out the environment for the lab a bit. To do so, I'm going to clone the existing Alpine VM.

First, I make sure I have the correct name for the existing VM with:

`sudo virsh list --all`, which yields:

```
 Id   Name          State
----------------------------
 4    alpine-base   paused
```

Next, I'll make sure to shut the VM down prior to cloning, then confirm it's off by repeating the above command.
`sudo virsh shutdown alpine-base`

However, this throws an error indicating a failure to shutdown 'alpine-base', as paused VMs cannot be shut down, but are also generally not safe to clone. Thus, I have to first resume then actually shut down the VM with:

```
sudo virsh resume alpine-base
sudo virsh shutdown alpine-base
```

Now, a list of my VMs properly displays `alpine-base` as shut off, and I can run the command to clone it.

```
sudo virt-clone \
	--original alpine-base \
	--name alpine-clone-1 \
	--file /var/lib/livbert/images/alpine-clone-1.qcow2
```
Success! Indicated by:
```
name alpine-clone-1 --file /var/lib/libvirt/images/alpine-clone-1.qcow2
Allocating 'alpine-clone-1.qcow2'                                                              | 4.0 GB  00:00:00     

Clone 'alpine-clone-1' created successfully.
```


