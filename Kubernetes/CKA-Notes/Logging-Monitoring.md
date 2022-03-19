---
title: Logging & Monitoring
description: 
published: true
date: 2022-03-19T15:50:41.664Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:50:39.676Z
---

Additional studies will be required for Prometheus

[Kubernetes+-CKA-+0300+-+Logging-Monitoring.pdf](:/f0a1714324984a28b661228183c31989)

# Monitoring

The point of monitoring is to get an idea of what is going on in your cluster. This could be at a POD level, or a a node level. This will monitor, store, and provice analytics for you.

Prometheus and Elastic Stack are two great Open Source options.

### Metrics Server

IN MEMORY only. This pulls data from the node, and pod, and stores the data in memory to be seen. The problem with this is that there is no Historical Data, as nothing is written to the disk. 

Kubelet is responsible for communicating with a metrics agent via cAdvisor.

`minikube addons enable metrics-server`

`git clone <metrics server repo>`\
`kubectl create -f <repo path>`

`kubectl top node` - this will show top level metrics for each node.

`kubectl top pod` - same as above, but for PODs

## Application Logs

`kubectl logs -f pod-name`\
-f streams the logs live

If there are more than 1 container in the POD, you need to specify the name of the container you want to view.