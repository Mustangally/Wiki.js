---
title: Scheduling
description: 
published: true
date: 2022-03-19T15:46:44.281Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:46:41.531Z
---

# Manual Scheduling

Every Pod definition file by default has a "nodeName" field.

The Scheduler will go through the pods to check for pods that DON'T have a nodeName set. If one is not present, the scheduler will set the nodeName, thus "scheduling" the POD, or assigning it to a Node.

To manually assign a POD to a node, you must simply set the "nodeName" as the NODE that you want the pod scheduled to. But this ONLY works during the POD creation.

Another way to do this is create a binding object, and specify a target node, then send a POST request to the POD in JSON format. This essentially mimicks the actions of a Scheduler. 

# Labels & Selectors

Labels and selectors are essentially ways to group and address different sets of applications based on a criteria. You can think of this similar to the properties you'd use to sort through things on an online shop.

This is done by adding labels to an item, and then using selectors to sort through the labels in your system.

In a POD definition file, you would put the labels under the METADATA section, as so:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: app1
	  function: front-end
spec:
  containers:
    - name: app
	    image: app
```

One the pod is created, you can query the pod using the labels by running:\
`kubctl get pods --selector app=app1`

You can also use selectors in a ReplicaSet to allow the replicaSet to manage that POD.
you would include this under the `spec` root, under `selector` > `matchLabels`

Services also use Selectors to connect to pods or Deployments, in the same syntax as a ReplicaSet.

## Annotations

An Annotation would be included under the `metadata` root on the same level as `labels`, and is typically used to store things like app version.

# Taint & Toleration

The basic concept of this is to prevent a scheduler from utilizing a node for workloads. Applying a Taint can mark the NODE as "don't use".

For an analogy, think of a Taint as "bug spray". When you apply a taint (or bug spray) it prevents the POD (or bug) from landing on the node (you). A Toleration however, can be applied to a POD (bug) to allow it to bypass the taint (bug spray).

Back to K8s, Taints and tolerations can restrict or allow PODs on different nodes. Another way to look at it is a Taint is like a filter. Only PODs with a Tolerance matching that filter can be scheduled on the node.

## Apply a Taint to a Node

`kubectl taint nodes node-name key=value:taint-effect`

There are 3 taint effect options.\
- `NoSchedule` Pods will not be scheduled
- `PreferNoSchedule` PODs will try not to be scheduled here, but not guarunteed
- `NoExecute` New PODs will not land here, and all existing PODs on the node will be "evicted" if they don't have a tolerance.

Example:\
`kubectl taint nodes node1 app=blue:NoSchedule`\
This would mean that any POD without a Tolerance for "blue" will not be scheduled on this node.

NOTE: adding a `-` to the end of the taint command will REMOVE the taint from that node.

## Apply Toleration to a POD

The toleration specification is added to the POD definition file under `spec` on the same layer as `containers`. The following would allow a POD to be scheduled on a node with the previous Taint applied.
```yaml
tolerations:
  -key: app
   operator: "Equal"
   value: "blue"
   effect: "NoSchedule"
```
NOTE: THE VALUES MUST BE IN DOUBLE QUOTES

Another good note is that just because you have a toleration applied to a POD, this does not mean the POD will always be scheduled on the node with the matching taint. This is done with "Node Affinity" (to be discussed later)

By Default a Taint is applied to the master node to not allow any PODs to the master node. This is best practice, as it prevents PODs from interfering with K8s admin processes.

# Node Selectors

The usecase for Node Selectors would be to designate different nodes for running more intensive tasks, or other criteria that you can determine.

To do this, add a `nodeSelector` spec at the same level as `containers`
```yaml
nodeSelector:
  size: Large
```
The designation `Large` is a key-value pair defined as a label on one of the nodes in the cluster.

`kubectl label nodes <node-name> <label-key>=<label-value>`\
example:\
`kubectl label nodes node-1 size=large`

What if we wanted to apply more criteria, like in a "Large or Medium" case, only excluding "small" nodes?

This CANNOT be done with NodeSelectors, only NodeAffinity

# Node Affinity

The purpose of Node Affinity is to ensure PODs only run on a particular node. Affinity provides further functionality over a Node Selector.

the `affinity` spec again goes at the same level as `containers` but is significantly more complex.
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
	    nodeSelectorTerms:
		    - matchExpressions:
			      - key: size
				      operator: In
					    values:
						    - Large
							  - Medium
```
You can also use the `NotIn` Operator to prevent PODs from going to nodes labeled "Small".

You can also use the `Exists` Operator to only allow the POD to be scheduled to nodes with a "Size" label applied. In this case, you would want to be sure you never set a "Small" label to a node in the cluster.

## No Matching Nodes

If for some reason the labels on the Node are changed, what will happen. Well if we look at the example yml above, the long string `requiredDuringSchedulingIgnoredDuringExecution` indicates that once the POD is running on the Node, it will not leave until it is destroyed, or moved because of some other action taken.

Other types of affinity are:\
- `preferredDurringSchedulingIgnoredDuringExecution`
- `requiredDuringSchedulingRequiredDuringExecution` #not yet included as of creation of this course.

# Resource Requirements and Limits

Resource Requirements and Limits can prevent PODs from being scheduled on a given node, or even from being Scheduled at all. Essentially, you can apply minimum resource requirements (Resource Requests), or maximum resource limits to a POD, so that it can be scheduled properly on a Node that is capable of supporting the workload.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: app1
	  function: front-end
spec:
  containers:
    - name: app
	    image: app
		  resources:
		    requests:
			    cpu: 1
				  memory: "256M"
```

1 Count of CPU is essentially 1 CPU thread. Think of this the same way you would for configuring VMs.

For Memory, you can use G,M,K, or Gi,Mi,Ki

The Default limit applied to PODs in K8s is 1 cpu, and 512Mi for memory.
Changing the "limits" section of your definition file can adjust these values.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: app1
	  function: front-end
spec:
  containers:
    - name: app
	    image: app
		  resources:
		    limits:
			    memory: "2Gi"
				  cpu: 2
```

When a limit is set, K8s will automatically throttle the process so that it does not overuse the CPU. 

However, going over the limit for memory will cause the POD to be terminated.

### Note on default resource requirements and limits

In the previous lecture, I said - "When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi". For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.

```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: mem-limit-range
    spec:
      limits:
      - default:
          memory: 512Mi
        defaultRequest:
          memory: 256Mi
        type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-limit-range
    spec:
      limits:
      - default:
          cpu: 1
        defaultRequest:
          cpu: 0.5
        type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/

References:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource

### A quick note on editing PODs and Deployments

Remember, you CANNOT edit specifications of an existing POD other than the below.

    `spec.containers[*].image`

    `spec.initContainers[*].image`

    `spec.activeDeadlineSeconds`

    `spec.tolerations`

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod <pod name>` command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

![8096c8634cf204d4fd7362f314ad2107.png](:/4e3f2b4b67ed4bd19e156869020ace45)

![abb32cee1156f85078d7c8d93a089124.png](:/5217a8e24cc24b1eb3437131c374600d)

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

`kubectl delete pod webapp`

Then create a new pod with your changes using the temporary file

`kubectl create -f /tmp/kubectl-edit-ccvrq.yaml`

2. The second option is to extract the pod definition in YAML format to a file using the command

`kubectl get pod webapp -o yaml > my-new-pod.yaml`

Then make the changes to the exported file using an editor (vi editor). Save the changes

`vi my-new-pod.yaml`

Then delete the existing pod

`kubectl delete pod webapp`

Then create a new pod with the edited file

`kubectl create -f my-new-pod.yaml`


Edit Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

kubectl edit deployment my-deployment

# Daemon Sets

Daemon Sets are somewhat similar to ReplicaSets, in that they assist in deploying multiple instances of a POD. Where they differ however, is that they ensure that one Replica is running on each node in the cluster. Any time a new node is joined, a new POD is deployed to that node.

Good use cases for this tool would be monitoring or logging agents. This will ensure that your monitoring always has access to each node and the processes running on it.

`kube-proxy` is an example of a default Daemon set in K8s. `Weave-net` is another example, as it employs an agent on each node in the cluster.

```yaml
apiVersion: apps/v1 <#needs to include "apps"#>
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
	    selector:
        matchLabels:
          app: monitoring-agent
		  template:
		    metadata:
			    labels:
				    app: monitoring-agent
				spec:
				  containers:
				    -name: monitoring-agent
					   image: monitoring-agent
```

Very similar to `replicaset` file.

`kubectl get deamonsets`

## How does it work?

While you could manually add `nodeName` to each POD so that it gets scheduled accordingly, DaemonSets actually use Node-Affinity and the default scheduler to accomplish it task.

# Static PODs

This would be a way for you to directly tell a node to run a certain POD. You would not need to go through the normal cluster masters with kubeapiserver or anything. You can run this on an isolated node should you desire.

To do this, you would place the pod definition file in the `/etc/kubernetes/manifests` directory on the node.

KUBELET will periodically check this directory and will deploy any PODs in that directory, as well as manage and update them as needed.

#### NOTE:

You can ONLY deploy PODs this way. You CANNOT create deployments or replicasets etc this way.

In the `kubelet.service` dfinition, you can actually set the manifests filepath. Or, you can set up another `kubeconfig.yaml`, and put a `staticPodPath` within that. If your cluster is setup with the kubeadm tool, it will be configured this way.

In thi case, you will not have access to kubectl as you are not connected to kube-api-server. For this reason, you will need to rely on docker commands to monitor any running pods. 

kubelet is capable of running static pods as well as the pods sent from api-server. The ones that are deployed by kublet will also show up in a `kubectl get pods` command. HOWEVER, kubectl will not be able to manage static pods in the same way it could normally. You MUST manage the POD directly on the node in the manifests directory.

A usecase for static pods may be things like the control plane itself, as this doesn't require the control plane to be functioning to run.