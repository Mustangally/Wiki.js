---
title: Proxmox & Cloud Images
description: 
published: true
date: 2022-03-21T17:32:34.380Z
tags: 
editor: markdown
dateCreated: 2022-03-21T17:29:54.120Z
---

## What is a Cloud Image?

You might run into these more often in an actual cloud environment like AWS of GCP, as most compute instances used by a hosted provider will utilize a "Cloud Image". So what does that mean? A cloud image is essentially a very lightweight and minimal version of an OS. Take Ubuntu for example. Ubuntu's cloud images are tiny, coming in at only ~500MB. These images also do away with the traditional Ubuntu Server installation GUI, instead allowing you to pass in parameters ahead of time (we'll get there) so that you can skip the installation phase, and go straight to configuration.

Once you have your template in Proxmox, you can simply clone it, define properties, assign an SSH key, and you're off to the races. You can take it a step further and utilize Ansible to further automate the configuration of your server, so that really this whole process becomes 95% automated from inception to operation. This can significantly cut down on the time you spend building servers, and allow you to get back to your testing and deployments.

## Where do I get Cloud Images?

Luckily Ubuntu makes this super easy. You can simply google "Ubuntu Cloud Images", and you'll find their repository of cloud images. That being said, I'll save you the time: http://cloud-images.ubuntu.com/

The cloud image you choose is of course up to you, but I'd recommend sticking to the most up-to-date LTS distribution, which at the time of writing is going to be the "Focal" 20.04 LTS version. ("Jammy" 22.04 LTS is set to release shortly, so I'll probably be testing that soon as well, but the process for this guide shouldn't change.)

## What is Cloud-Init?

Cloud-Init in Proxmox is essentially a program that runs alongside the VM to supply configuration variables to the OS. You can define things like username & passwords, SSH Keys, and network parameters. This saves you the time of sitting through the installation process for an OS, because you simply fill out what you need ahead of time, and Cloud-Init handles the passthrough of that data for you.

You can add a Cloud-Init drive to a VM in Proxmox via the 'device' tab for the VM in question, or you can do it via CLI (which we'll see later). It attaches to the VM as an ISO image via a virtual CDROM device. You will need to keep this attached to your VM, so remember that there may be some restrictions caused by this (like migrations).

## How to Put This All Together

Ok so we have our Image (if not, go download that to your Proxmox Server however you wish), and we understand what a Cloud-Init drive is. Now what? Well lets work on putting this all together to see how it works in practice. We're going to do all of this via CLI because I think it provides better visibility into what we're actually doing, but once we have that understanding you can do this in the Proxmox GUI as well should you choose to do so.

Ok so first things first, open a terminal session to your Proxmox host either via the GUI or SSH. Now we'll go ahead and create a new VM:

`qm create 200 --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0`

This is creating a VM named "ubuntu-cloud" with an ID tag of 200, 2GB of RAM, 2 CPU Cores, and a network interface attached to the host bridge network "vmbr0"

Next, we'll attach our Cloud Image to the VM on the 'scsi0' interface:

`qm set 200 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-200-disk-0`

No we can add the Cloud-Init Drive on the 'ide2' interface:

`qm set 200 --ide2 local-lvm:cloudinit`

We'll want to make the cloud-init drive bootable and restrict BIOS to boot from disk only:

`qm set 200 --boot c --bootdisk scsi0`

And lastly we're going to add a serial console:

`qm set 200 --serial0 socket --vga serial0`

**DO NOT start your VM yet. Doing so will generate unique identifiers for the machine which can cause network identification conflicts down the line with clones.**

Now that everything is set up for your template, you can double check your configurations (CPU, RAM, etc.), Configure the Cloud-Init Parameters, and then convert to template. If you don't know how to do that, click on the VM you want to convert, then click "more" in the upper right hand corner of the GUI:

If you want to expand your hard drive you can on this base image before creating a template or after you clone a new machine. This is entirely up to you, and I'd recommend setting the template to be the minimum default spec you can see yourself utilizing. Clones can always be upgraded when you create them.

## Create Clones!

Now that you've created your template, you can start making clones! Hopefully you found this guide useful, and it saves you time in the future!


## Troubleshooting

Incase you accidentally started your VM prior to converting it to a template, you can simply run the following to reset:
```bash
sudo rm -f /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
```
Then shutdown the machine.
