---
title: Core Concepts
description: 
published: true
date: 2022-03-19T15:45:27.999Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:45:25.801Z
---

This page will address the core concepts in K8s. Later pages will deep dive into given topics, but this is a general overview of K8s and how different components work.

[Kubernetes+-CKA-+0100+-+Core+Concepts.pdf](https://wiki.kkovach.com/attachments/5)

[kubernetes-services-updated.pdf](https://wiki.kkovach.com/attachments/6)

# Cluster Architecture

## NODES

### Worker Nodes

- run the containers for the cluseter
- Container runtime (Docker)
- kube-proxy - ensures container communication
- kubelet - agent on each node that recieves instructions from the apiserver
    - also sends health checks to apiserver

### Master Node

Manages, plans/schedules, monitors, and stores data on containers & nodes in the cluster

- Control Plane Components
    - ETCD - Key Value Store
        - Stores all of the info of the cluster
            - Nodes
            - Pods
            - Configs
            - Secrets
            - Accounts
            - Roles
            - Bindings
    - kube-scheduler - Identifies requirements of deployments/containers
    - Controllers - Monitor's deployments and services
        - Node controller
        - Replication Controller
    - kube-apiserver - Orchestrates communication internal and external for the cluster
    - Container runtime (Docker)
    - kubelet - agent on each node that recieves instructions from the apiserver
        - also sends health checks to apiserver

## ETCD

### What is ETCD

ETCD is a distributed, reliable key-value store

### Key-value store

Unlike a traditional tabular or relational database, data is stored as a "key" and a "value" (name: "John Doe")

### Install ETCD

1.  Download Binaries
2.  extract
3.  run executuable

- listens on port2379

### Operate ETCD

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3. By default its set to use Version 2. Each version has different sets of commands.
For example ETCDCTL version 2 supports the following commands:

```
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

Whereas the commands are different in version 3

```
etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put
```

To set the right version of API set the environment variable ETCDCTL_API command
`export ETCDCTL_API=3`

When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.

Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

```
--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key
```

So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

```
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 
```

### ETCD in K8s

`kubectl` pulls data from the etcd server

## kube-apiserver

When you make a command like `kubectl get pods` you're sending a request through the apiserver to ETCD, and the response follows the same path. All data between different services within a cluster pass through the apiserver in between each service. ("k8s the hard way" will provide a much deeper understanding of this communication flow path)

To view the current kube-apiserver settings in an existing cluster, look in `cat /etc/kubernetes/manifests/kube-apiserver.yml` or run `ps -aux | grep kube-apiserver`

## Kube-Controller-Manager

These controllers monitor status of different part of the cluster. By Default, all managers are used

- Node-controller (every 5 seconds)
- Allows 40s before marking a node as "unreachable"
- After 5min, it will remove all of the pods scheduled for the node, and migrate them elsewhere
- Replication Controller
- handles pod creation per replicaset settings

To view the current kube-controller-manager settings in an existing cluster, look in `cat /etc/kubernetes/manifests/kube-controller-manager.yml` or run `ps -aux | grep kube-controller-manager`

## kube-scheduler

Only responsible for assigning pods to a node. It does NOT actually place the pod on the node.
 This is needed so that pods are organized and spread amongst nodes appropriately
To schedule pods on a node, the scheduler goes through a short list of tasks;
1. Filter Nodes
	- this filters out nodes that will not meet the pod's resource requirements
2. Rank Nodes
	- Calculates remaining resources after pod placement, and ranks the node accordingly (more is better)

To view the current kube-scheduler settings in an existing cluster, look in `cat /etc/kubernetes/manifests/kube-scheduler.yml` or run `ps -aux | grep kube-scheduler`

## kubelet

This can be compared to the "captain" of each node. It is responsible for:
1. registering nodes
2. Creating Pods
3. Monitoring Nodes & Pods
4. reporting to apiserver

## kube-proxy

This deploys a "pod network" that allows inter-pod networking. For example, a webapp communicating with a database.

Note, pod IP's are not constant so services are used to identify pods. Services however are not on the "pod network", so Kube-proxy allows the service to communicate on the nodes and between pods.
- IP tables are used, and maintained by the kube-proxy to handle routing within the cluster

# PODs

The primary goal of K8s is to deploy containers on the worker nodes of a cluster. However, K8s does not deploy containers directly onto the node. Instead, it uses PODs. These encapsulate the containers, creating a space for each container to run. This is a single "instance" of an application, and is the smallest object you can create in K8s

When you need to scale an application, you do not deploy a container within an existing POD. Rather, you deploy additional PODs, or instances, in the cluster to match requirements. Typically have a 1 to 1 relationship with containers, however you CAN run multiple containers within a POD.
- Multi-Container PODs most often contain a primary application, and "helper" containers like databases. This allows apps and helpers to be scaled together, and communicate directly on the "localhost" address.

### Example of Running a POD

```
kubectl run nginx --image nginx
kubectl get pods #this will show running PODs
```

## PODs with YAML

K8s uses YAML files as the template for creating PODs (as well as other objects that will be addressed later)
All of them follow a consistent core template. An example POD YAML file would be:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  labels:
    app: example
    type: front-end
spec:
  containers:
    - name: example-container
      image: example
```

`kubectl create -f pod-definition.yml` \- creates the pod as defined in the file (-f designates "file")\
`kubectl describe pod example-pod` \- Shows details of the running pod

# ReplicaSets

Replication-Controller monitors and manages the number of replicas of a given POD. Maintains a specified number of PODs. The Controller is capable of spanning nodes in the cluster, and will distribute PODs in an optimal fashion.

- This allows for load balancing and quick & easy scaling of an application.

ReplicaSets are replacing Replication-Controllers. This is the NEW technology for managing replicas. Moving forward ReplicaSets are the assumed tool.

## Editing Existing ReplicaSets without YAML

`kubectl edit replicasets.apps new-replica-set`

Example ReplicaSet YAML file:

```yaml
apiVersion: apps/v1 <#needs to include "apps"#>
kind: ReplicaSet
metadata:
  name: example-replicaSet
  labels:
    app: example
    type: front-end
spec:
  template:
    metadata:
        name: example-pod
        labels:
            app: example
            type: front-end
        spec:
          containers:
            - name: example-container
              image: example
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

`kubectl create -f replicaset.yml`\
`kubectl get replicaset`

### Selectors

The "selector" definition will direct the ReplicaSet toward the pods you want it to manage. This can be pre-existing pods, or yet-to-be created pods. If there are already existing PODs, the ReplicaSet will monitor and replace PODs as needed.
This is one of the primary differences from a Replication-Controller.

To match a ReplicaSet to a POD, you need to make sure the selector is pointing to one of the labels defined in the POD.

## Scaling with ReplicaSets

There are multiple ways to update the number of replicas in a ReplicaSet.

1.  The most obvious initially is to simply modify the .yml file to reflect the number of replicas you want.
    \-\- you will then run `kubectl replace -f replicaset.yml` to update the running ReplicaSet
2.  Alternatively, you can run `kubectl scale --replicas=6 replicaset.yml`
3.  Or, `kubectl scale --replicas=6 replicaset`
    \-\- This option will not update the definition file, this means if you delete and re-create the replicaset, your replicas will be reset to original specification.

# Deployments

Deployments allow you to easily manage all aspects of application deployment. This is the "largest" object you can deploy in K8s. It encapsulates containers, pods, & replicasets.

It can manage:

- replicas
- versioning (with rollouts/rolling updates & rollbacks)
- edit the apps in the deployment
- pause & start deployments

Example Deployment Definition File:

```yaml
apiVersion: apps/v1
kind: Deployments
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
        name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
    replicas: 3
    selector:
      matchLabels:
        type: front-end
```

check work:\
`kubectl create -f deployment.yml`\
`kubectl get deployments`\
`kubectl get replicaset`\
`kubectl get pods`

# Namespaces

You can think of namespaces as being similar to a family name. People within the family will address others by their first name.  But someone who belongs to a different family will refer to an individual as their first AND last names.

In the same way, in K8s, the "family name" is the namespace. PODs will belong to different namespaces within the cluster. Within a namespace, PODs can address eachother directly by their names. But to communicate with a POD in another namespace, it needs to append the namespace name to the POD name. an example would be `db-service.dev.svc.cluster.local` to communicate with the db-service POD.

Often times namespaces are not necessary in smaller deployments like test environments, but in a larger deployment, this organization becomes valuable.

Defaults would be:
- Default
- kube-system
- kube-public

Benefits of using namespaces in a cluster would be:
- Isolation of resources & policies
- Resource quota's per namespace

the `kubectl get pods` command by default only addresses the "default" namespace. to address others, you will need to add `--namespace=name` flag.

to specify the namespace for a pod in a yaml file, add it into the "metadata" section on the same level as "name" or "labels"

## Creating a Namespace

```yaml
apiversion: v1
kind: Namespace
metadata:
  name: dev
```
`kubectl create -f namespace.yml`

Alternatively:\
`kubectl create namespace name`

## Change the context of Kubectl

You can change the context of Kubectl to address a different namespace than default. To do this:\
`kubectl config set-context $(kubectl config current-context) --namespace=name`

## Resource Quota

compute-quota.yaml
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
	  requests.cpu: "4"
	  requests.memory: "5Gi"
	  limits.cpu: "10"
	  limits.memory: "10Gi"
 ```
 
 # Services
 
Services are the primary utility used for communication between groups of pods or microservices in the cluster. This may be between the deployment and external users, or to other internal resources like databases.
 
Without a service, there would be no way for an external user to access the pods inside of a cluster. If we remember how internal networking for the cluster works, each node and pod is assigned an IP range to operate within. If your IP is 192.168.1.10, and the pod has an internal IP of 10.244.0.2, there would be no way for you to directly access that pod. While you could in theory SSH into the node that pod lives on, then access the POD, but that is not a user-friendly way of accessing applications deployed in K8s.

A service is then used to provide an interface on the NODE to access the POD. For example, a "NodePort" configuration would map a POD port to a port on the node, allowing you to access the pod on the node's IP address.

### 3 Types of Service

1. NodePort
	- Service makes a port on the POD availiable on a port on the NODE
2. ClusterIP
	- Service creates a virtual IP inside of the cluster to allow communication between deployments in the cluster, like front-end servers communicating with back-end servers.
3. LoadBalancer
	- This allows external access to the cluster by addressing an IP address for the whole cluster. The LoadBalancer then directs traffic to the appropriate application running on in the cluster.

## NodePort

As stated before, a "NodePort" configuration would map a port on a POD to a port on the node, allowing you to access the pod on the node's IP address.

There are 3 different ports that need to be addressed in a NodePort configuration.
These are:
 - Target Port: This is the port on the POD, or where the "destination" is.
 - Port: The port on the service itself. This is because the service is basically a server of it's own, which has it's own IP
 - NodePort: This is the port on the NODE. NodePorts can only be between 30000-32767

```yaml
apiVersion: v1
kind: Service
metadata:
	name: myapp-service
spec:
	type: NodePort
	ports:
		- targetPort: 80
			port: 80
			nodePort: 30008
	selector:
		app: myapp
		type: front-end
```

`kubectl create -f service-definition.yml`\
`kubectl get services`
 
The selector section with the defined labels below it, are what allow the service to connect to a specific deployement, and distribute requests between different replications of the app. This uses a random algorithm to distribute traffic.
 
 ## ClusterIP
 
 A clusterIP is a service that stands between different layers of a deployment. For example, if you have a front-end, back-end, and a db all working together to provide an application. Typically, you will have multiple replicas of each layer of that stack. To enable each pod to communicate with the different layers without having to address a specific pod, you use a ClusterIP. This allows the pod to talk to one point, and access any of the supporting pods that are on the other side of the service.
 
 ```yaml
apiVersion: v1
kind: Service
metadata:
	name: back-end
spec:
	type: ClusterIP #this is the default service, so the "type" field is not technically required
	ports:
		- targetPort: 80
			port: 80
	selector:
		app: myapp
		type: back-end
```

`kubectl create -f service-definition.yml`\
`kubectl get services`

 ## LoadBalancer
 
This allows external access to the cluster by addressing an IP address for the whole cluster. The LoadBalancer then directs traffic to the appropriate application running on in the cluster.
 
 On a cloud platform like AWS of GKE, you can leverage the native load balancer for those providers to access your kubernetes cluster. If you are not on a cloud provider however, you would need to install something like metalLB and traefik to accomplish this.
 
# Imperative vs Declarative
 
In the IAAS world, you have two approach options to deploying services and apps.\
These are: Imperative, and Declarative

## Imperative Approach

In the Imperative approach, you are defining what AND how to do things.

In terms of K8s, an example would be a set of step by step instructions as such:\
- Provision a VM
- Install NGINX
- Edit the config to expose 8080
- edit config to define file path
- load wepages from a GIT repo
- Start the NGINX server

## Declarative Approach

Unlike the imperative approach, the Declarative approach only requires that you define WHAT you want to happen. You are not concerned with HOW.

For example, you simply define the end product you want:\
```
VM Name: web-server
Package: nginx
Port: 8080
Path: /path/to/example
Code: GIT Repo - x
```

Once this is defined, the system is reponsible for completing the tasks required to produce the desired end-goal.

This is where tools like Ansible, Terraform, or Chef come into play. These tools automate the configuration process based on a definition of the end product that you provide.
 
# Certification Tips

As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the `kubectl run` command can help in generating a YAML template. And sometimes, you can even get away with just the `kubectl run` command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises.

Reference (Bookmark this page for exam. It will be very handy):
[https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)

Create an NGINX Pod\
`kubectl run nginx --image=nginx`

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)\
`kubectl run nginx --image=nginx --dry-run=client -o yaml`

Create a deployment\
`kubectl create deployment --image=nginx nginx`

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)\
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)\
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.\
`kubectl create -f nginx-deployment.yaml`\
OR\
In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.\
`kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

## Imperative Commands with Kubectl

While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

`--dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run=client option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

`-o yaml`: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

### POD
Create an NGINX Pod

`kubectl run nginx --image=nginx`

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

`kubectl run nginx --image=nginx --dry-run=client -o yaml`

`kubectl run nginx --image=nginx --port=8080`

### Deployment
Create a deployment

`kubectl create deployment --image=nginx nginx`

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Generate Deployment with 4 Replicas

`kubectl create deployment nginx --image=nginx --replicas=4`

You can also scale a deployment using the kubectl scale command.

`kubectl scale deployment nginx --replicas=4`

Another way to do this is to save the YAML definition to a file and modify

`kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`

You can then update the YAML file with the replicas or any other field before creating the deployment.

### Service
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`\
(This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

Reference:
[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

[https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)
