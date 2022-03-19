---
title: PCIE Passthrough to VMs
description: 
published: true
date: 2022-03-19T20:09:06.337Z
tags: proxmox
editor: markdown
dateCreated: 2022-03-19T20:09:04.430Z
---

# TrueNAS on Proxmox - How to Passthrough Drives to VM


## Passthrough full PCI Controller

The easiest way to pass drives through to a VM is by assigning a full PCI-E controller to the VM, and attaching the drives to the controller. To attach the controller, click on the VM, go to "Hardware", click the dropdown menu for "Add", and select PCI Device

![image-1643343088052.png](/image-1643343088052.png)

## Passthrough Drive without a PCI Controller

In the event you do not have a PCI controller to passthrough to the VM, you will need to individually assign the drives to the VM via SSH shell to your Proxmox host.

### List the Available Drives
First you'll need to list the drives available to your system. To do this, run:
```bash
lsblk -o +MODEL,SERIAL
```
You can also simply navigate to `/dev/disk/by-id` to list drives and partitions

![image-1643343648014.png](/image-1643343648014.png)
![image-1643343590861.png](/image-1643343590861.png)

## Add Drives to VM

Now that you finally have the list of drives that you want to add to your VM, you just have to run the following command for each drive, replacing the model & serial number to match the drive you're addressing, and the connector type/sequencing number (scsi1,2,3,etc)

```bash
qm set 103 -scsi1 /dev/disk/by-id/DISKNAME
```

![image-1643343936718.png](/image-1643343936718.png)