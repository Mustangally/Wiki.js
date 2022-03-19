---
title: Networking Pre-Reqs
description: 
published: true
date: 2022-03-19T15:56:56.057Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:56:54.010Z
---

[Kubernetes+-CKA-+0800+-+Networking-v1.2.pdf](:/f25cb1a5124d4d36a1500678047ba5be)

[Ingress.pdf](:/caa1f28ee2b74607a3cf7756e7e7efe3)

# Switching & Routing

## Take Away Commands

`ip link` - list and modify interfaces
`ip addr` - see IP Addresses on Interfaces
`ip addr add` - set IP address on an interface

Persist the above by modifying `/etc/networkinterfaces`

`ip route` - Check the current routes
`ip route add` - add a new route
`cat /proc/sys/net/ipv4/ip_forward` - check the status of ip forwarding for router operations

# DNS

DNS resolution is what relates a readable name to an IP address.

You can add DNS entries to your linux host by modifying the `/etc/hosts` file, and adding the IP and DNS Name.
You can also instead add a DNS Server to your system, so that it looks there for any DNS requests.
Do this by editing `/etc/resolv.conf`
**NOTE** that the `/etc/hosts` file will always overrule the DNS server, UNLESS you modify the `/etc/nsswitch.conf` file to re-order this process.

Within the `resolv.conf` file you can add both a nameserver, or a search. A search allows you to specify a domain name, and any simple subdomain requests will first be searched against the domains in the search row.

# CoreDNS
In the previous lecture we saw why you need a DNS server and how it can help manage name resolution in large environments with many hostnames and Ips and how you can configure your hosts to point to a DNS server. In this article we will see how to configure a host as a DNS server.

We are given a server dedicated as the DNS server, and a set of Ips to configure as entries in the server. There are many DNS server solutions out there, in this lecture we will focus on a particular one – CoreDNS.

So how do you get core dns? CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.
![c03e7023646a491d730e480aec4039cf.png](:/e83c45f313674de29a3865991a17ccfb)
Run the executable to start a DNS server. It by default listens on port 53, which is the default port for a DNS server.

Now we haven’t specified the IP to hostname mappings. For that you need to provide some configurations. There are multiple ways to do that. We will look at one. First we put all of the entries into the DNS servers /etc/hosts file.

And then we configure CoreDNS to use that file. CoreDNS loads it’s configuration from a file named Corefile. Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the Ips and names from the /etc/hosts file on the server.
![0a0c8a79680c6af93d9c9bc085107132.png](:/33b71796dc2a404f8fad3d1f776d803a)
CoreDNS also supports other ways of configuring DNS entries through plugins. We will look at the plugin that it uses for Kubernetes in a later section.

Read more about CoreDNS here:
https://github.com/kubernetes/dns/blob/master/docs/specification.md
https://coredns.io/plugins/kubernetes/

# Network Namespaces

Think of namespaces like the rooms in a house. The house is your network as a whole. It is a way of segmenting your network and isolating them, although you can enable communication between rooms or namespaces should you wish. While you're within a namespace, you can only see resources and processes that are within your namespace.

Like more broad networks, your namespace can also have an interface that it communicates through. It can also have its own routing and ARP tables that it uses. 

To create a namespace in Linux, you can run:
`ip netns add <name>` then view them with `ip netns`

To list interfaces on the host, or any other network command for that matter, you would again run `ip link` for example. But for a NS, you can run `ip netns exec <name> ip link` to exec into the namespace. Or, the simpler way, is to run `ip -n <name> link`

If you want to enable communication between two namespaces, you can add a pipe or "virtual cable" between them. Do this by running `ip link add veth-<name1> type veth peer name veth-<name2>`. Then you link the interfaces to a namespace by running `ip link set veth-<name1> netns <name1>` and same for NS 2. Now you can assign an IP to the Namespace with `ip -n <name1> addr add <IP> dev veth-<name1>`. Set the interface to "UP" with `ip -n <name1> link set veth-<name1> up`

## Create an Internal Network, or Linux Bridge

First you need to create a new virtual interface for the host. Run `ip link add v-net-0 type bridge`, then `ip link set dev v-net-0 up`\
Now you need to connect the NS Interfaces to the virtual switch. To do this, `ip link add veth-red type veth peer name veth-red-br` to connect the "Red" NS interface to the bridge interface. You'll then need to attach with `ip link set veth-red netns red` & `ip link set veth-red-br master v-net-0`\
At this point, assign IP addresses again and turn the interface "UP"

# Docker Networking

`host network`, when you use this network the container can be reached at the host's IP Address and a given port. Note that you cannot have two containers running on the same port this way.

`bridge network` this is an internal private network to the host. Containers can communicate together, but they are not reachable from an external location. When you start a container in bridge mode, a new namespace is created for that container. Every container has it's own namespace. For the remainder of this section, the "container" and the "namespace" are one and the same.
Docker automatically creates a interface on both the container and the host's bridge network, and a v-cable between them. 

## Port Mapping

So now that we have the containers on a bridge network isolated within the host, how do we access them? Docker provides a port mapping option that connects a port on the container to a port on that host. This is similar to "Host Network" except that only the specific ports are reachable, and you can swap the ports like 80 to 8080.

# CNI (Container Networking Interface)

A CNI is essentially a set of standards to define how container networking should be organized and configured. This allows orchestrators like K8s to be able to run a program or script to setup the network for a new container, regardless of the underlying runtime. It ensures compatibility between runtimes as they all configure networks the same way.

On the runtime side:
1. Container Runtime should create a namespace
2. Identify the network to attach to
3. Runtime should invoke the network plugin when added
4. Runtime should invoke the network plugin when deleted
5. JSON format for networking configs.

On the Plugin side:
1. Must support cmd line arugments (add/del/check)
2. must support parameters container ID, network ns, etc.
3. must manage IP address assignment
4. return results in a spcific format

**NOTE** that docker does not use CNI, but they do use a different proprietary set of standards. If you wanted to use a CNI plugin with docker, you would have to invoke the plugin manually. This is what Kubernetes does when it creates a container. It creates the container without a network, then invokes the plugin to create it.

# Cluster Networking

Hosts much each have a unique IP and MAC

Port Reqs:
- kubeAPI: 6443
- kubelet: 10250
- kube-scheduler: 10251
- controller-manager: 10252
- Services: 30000-32767
- ETCD: 2379
- ETCD for HA: 2380

# Important Note about CNI and CKA Exam

An important tip about deploying Network Addons in a Kubernetes cluster.

In the upcoming labs, we will work with Network Addons. This includes installing a network plugin in the cluster. While we have used weave-net as an example, please bear in mind that you can use any of the plugins which are described here:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model

In the CKA exam, for a question that requires you to deploy a network addon, unless specifically directed, you may use any of the solutions described in the link above.

However, the documentation currently does not contain a direct reference to the exact command to be used to deploy a third party network addon.

The links above redirect to third party/ vendor sites or GitHub repositories which cannot be used in the exam. This has been intentionally done to keep the content in the Kubernetes documentation vendor-neutral.

At this moment in time, there is still one place within the documentation where you can find the exact command to deploy weave network addon:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node (step 2)