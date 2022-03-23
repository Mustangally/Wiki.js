---
title: Can't Access Proxmox GUI Through NGPM
description: 
published: true
date: 2022-03-23T19:25:34.871Z
tags: proxmox, networking, reverse-proxy
editor: markdown
dateCreated: 2022-03-23T19:25:34.871Z
---

<p id="bkmrk-so-you-have-nginx-pr">So you have Nginx Proxy Manager setup to remotely access your Proxmox server. Everything is working fine <em>accept</em> your virtual console? This is a common issue and I have an easy fix!</p>
<p id="bkmrk-step-1%3A-edit-the-pro">Step 1: Edit the Proxy Host</p>
<p id="bkmrk-"><img src="https://snip.lol/JipE3/jAkoHoyu71/raw.png"></p>
<p id="bkmrk-turn-on-the-switch-f">Turn on the switch for "Websockets Support" then save it.</p>
<p id="bkmrk-step-2%3A-done.">Step 2: Done.</p>
<p id="bkmrk-as-always%2C-please-ex">As always, PLEASE exercise caution when exposing this over the internet. At the very least, enable Proxmox TFA and add another layer of security to your Proxmox host.</p>

You should note that I DO NOT expose proxmox to the internet, but I do have an internal reverse proxy for *.local.kkovach.com DNS entries on my LAN DNS Server