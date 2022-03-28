---
title: Building an HA K3S Cluster with Ansible
description: 
published: true
date: 2022-03-28T23:58:42.941Z
tags: kubernetes, k3s, ansible, automation
editor: markdown
dateCreated: 2022-03-27T21:14:47.378Z
---

# Prep
First, you’ll need Ansible installed. See my page on installing [Ansible](/Automation/Ansible-Basics) if needed.

Next, you’ll want to clone [this repo](https://github.com/Mustangally/k3s-ansible) to your machine so that you can make changes to fit your environment.

Should you wish to read a more in-depth explanation of what is going on, please see my [blog post](https://www.kkovach.com/building-a-k3s-cluster-with-ansible/).

# Installing k3s
Next, you’ll want to copy the `sample` directory within the `inventory` directory.
```bash
cp -R inventory/sample inventory/my-cluster
```

### Edit the Hosts File
Next, edit the `inventory/my-cluster/hosts.ini` to match your systems. DNS works here too.
```
[master]
10.0.0.110
10.0.0.111
10.0.0.112

[node]
10.0.0.121
10.0.0.122

[k3s_cluster:children]
master
node
```

### Edit Variables
Edit `inventory/my-cluster/group_vars/all.yml` to your liking. See comments inline.

> Note: These are for an advanced use case. There isn’t a one size fits all setting for everyone and their needs, I would try using k3s without these before changing. This could have undesired effects like nodes going offline, pods jumping or being removed, etc… This might come at the cost of stability
{.is-info}

```
extra_server_args: "--no-deploy servicelb --no-deploy traefik --write-kubeconfig-mode 644 --kube-apiserver-arg default-not-ready-toleration-seconds=30 --kube-apiserver-arg default-unreachable-toleration-seconds=30 --kube-controller-arg node-monitor-period=20s --kube-controller-arg node-monitor-grace-period=20s --kubelet-arg node-status-update-frequency=5s"
extra_agent_args: "--kubelet-arg node-status-update-frequency=5s"
```

### Modify K3S Arguments
It’s best to start using these args, and optionally include traefik if you want it installed with k3s

```
extra_server_args: "--no-deploy servicelb --no-deploy traefik"
extra_agent_args: ""
```

### Run the Playbook
Start provisioning of the cluster using the following command:
```
ansible-playbook site.yml -i inventory/my-cluster/hosts.ini
```

# Remove the Cluster but Retain Nodes

To remove k3s from the nodes:
```
ansible-playbook reset.yml -i inventory/my-cluster/hosts.ini
```

# kube config

To get access to your Kubernetes cluster and copy your kube config locally run:
```
scp debian@master_ip:~/.kube/config ~/.kube/config
```

# Testing your cluster

Be sure you can ping your VIP defined in inventory/my-cluster/group_vars/all.yml as apiserver_endpoint
```
ping 192.168.30.222
```

## Getting nodes
```
kubectl get nodes
```

## Deploying a sample nginx workload
```
kubectl apply -f example/deployment.yml
```

## Check to be sure it was deployed

kubectl describe deployment nginx

## Deploying a sample nginx service with a LoadBalancer
```
kubectl apply -f example/service.yml
```
Check service and be sure it has an IP from metal lb as defined in inventory/my-cluster/group_vars/all.yml
```
kubectl describe service nginx
```
## Visit that url or curl
```
curl http://192.168.30.80
```
You should see the nginx welcome page.

## Cleanup
You can clean this up by running
```
kubectl delete -f example/deployment.yml
kubectl delete -f example/service.yml
```