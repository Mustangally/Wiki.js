---
title: PfSense in Proxmox
description: PfSense Configuration to hide VMs behind a double NAT
published: true
date: 2022-05-01T07:33:50.109Z
tags: proxmox, networking
editor: markdown
dateCreated: 2022-05-01T07:33:50.109Z
---

# Intent
My intent with this experiment is to install a PfSense instance infront of my VM's hosted in Proxmox, so that I can more efficiently control vlans and access to and between my VM's. I don't feel that my cheap router that runs my primary LAN is adequate, so this is also a proof of concept for the future, wherein I can deploy PfSense in a "Production" setting as my primary router for the whole household.

# Getting Started

First of all we need to get PfSense running. Create a new VM, assinging the proper hardware resources, and booting the PfSense Image.

The important part to note here is the networking configuration for the VM. We're going to be attaching the primary bridge network for Proxmox, as well as a second interface we've created.
![proxmox_network_interfaces.png](/proxmox_network_interfaces.png)
> Note that "vmbr0" is linked to port "eno1". This is the only physical interface avaliable to this proxmox node. "vmbr1" does not need a physical port, as it will be the "LAN" interface that connects to the VM's.
{.is-info}

## Post Install Setup

Boot the PfSense VM, and you can walk through the installation. For the most part, you can leave the install settings to default.

Once PfSense reboots, you'll want to select the correct interfaces for your WAN and LAN connections. Make sure these match up to the Proxmox interfaces.

#### Test your WebUI

You're going to need to temporarily disable the firewall to allow access from the WAN side to your WebUI. TO do this, select option "8" in PfSense to open a shell, and type `pfctl -d`. Be aware this is disabling the firewall entirely, so this is not secure at all. To renable it later, we'll use `pfctl -e`.

Once your web GUI is working, you can change your WAN interface from DHCP to static. Select option "2", and enter the appropriate information.

## WebUI Initial Setup

This part is fairly staightforward. First we'll login with the credentials `admin`, `pfsense`.

Once we're in, we'll be brought to a setup checklist. Click through here and make sure nothing stands out. The big thing will be setting up your "LAN" IP address. This is the IP your VM's will be looking for as their gateway.

