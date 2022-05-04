---
title: Backup TrueNAS Scale to Azure Blob
description: 
published: true
date: 2022-05-04T16:32:13.092Z
tags: storage, azure, truenas
editor: markdown
dateCreated: 2022-05-04T16:32:13.092Z
---

# Creating a TrueNAS Cloud Backup Job to Azure
TrueNAS is my NAS operating system of choice at home, and it does a great job of managing and protecting data. However I do have data that is "irreplacable", that I'd like to have backed up off-site per the 3-2-1 backup model. This guide will be a step-by-step to backup TrueNAS datasets to Azure Blob Storage.