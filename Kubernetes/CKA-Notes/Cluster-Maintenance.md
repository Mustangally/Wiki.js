---
title: Cluster Maintenance
description: 
published: true
date: 2022-03-19T15:53:46.212Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:52:57.383Z
---

[Kubernetes+-CKA-+0500+-+Cluster+Maintenance-v1.2.pdf](:/bd195fdc4f4044539566e4b78881ffd9)

# OS Upgrades

Sometimes you may need to pull down a cluster node to perform OS upgrades or patches to the node itself. What do we do in this event?

Well, when one node goes down, all of the PODs on that node are no longer avaliable. The k8s will by default allow a 5min buffer before it begins to move PODs around to different nodes. At this point, the PODs are terminiated and the ReplicaSet will recreate the POD on a new node.

You can change the timout timer with `kube-controller-manager --pod-eviction-timemout=5m0s`

Once a node comes online, it will not have any PODs. Now that it is back, the kube-scheduler will schedule PODs on this node as normal.

The safe way to do this without relying on the timeout, is to `kubectl drain node-1` to tell k8s to move the running PODs to a new node, so that you can take down the node without impacting any running services. It does this as you would expect, by terminating the PODs on the node selected, and re-creating them on other nodes in the cluster.

**note:** When you run the `drain` command, the node is marked as "un-schedulable", so that PODs are not recreated on this node. you will need to run a `kubectl uncordon node-1` to make it availiable again once you are done performing maintenance on said node. 


`kubectl cordon node-1` will mark a node as unschedulable however it does not terminate any running PODs. It only prevents new PODs from being deployed.

# Kubernetes Releases/Versions

We know already that when we create a cluster, we are installing a certain api-version of K8s. This is seen in the `kubectl get nodes` output. But from time to time, we need to update this version as new versions are released. 

As K8s is updated, it goes through a standard software dev process. In between releases, you will have alphas and betas, before it merges back in to the main branch as a stable release. The alphas and betas are meant to test and patch new features.

It is imporant to note that not every component in a release may be on the same version. ETCD Cluster and CoreDNS for example are seperate projects, but still get bundled into a K8s release.

[https://kubernetes.io/docs/concepts/overview/kubernetes-api/](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)

Here is a link to kubernetes documentation if you want to learn more about this topic (You don't need it for the exam though):

[https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)

[https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md)

# Cluster Upgrade Process

In Terms of compatibility, nothing can be at a higher version than Kube-apiserver, as this is the communication process between all of the different components.

controller-manager and kube-scheduler however can be 1 version behind the api-server, and kubelet and kube-proxy are allowed to be 2 versions behind api-server. 

The Kubectl utility is the only thing that breaks this rule. It can be within 1 version in either direction to work properly. This is to allow live upgrades to your cluster.

Kubernetes support structure will only go back 3 minor release versions. This means that if you want to stay with a "supported" version, you need to be on one of the 3 most recent releases. This should govern your update schedule. You will want to stay WITHIN the supported range.

The recommended approach is to upgrade 1 version at a time, and not to version hop. For example, from 1.11 to 1.12 to 1.13, not from 1.11 to 1.13.

Cloud service will typically have an upgrade utility to assist you in this process, or if you used Kubeadm to create the cluster, you can run `kubeadm upgrade plan` and `kubeadm upgrade apply`. To do this from scratch, you will need to do it manually 1 component at a time.

## Upgrade Process Order

The first thing to happen when upgrading a cluster is to upgrade the master nodes, THEN the worker nodes.

**note** that when the Master node goes down, your management utilities that tie into kube-api-server, and management processes like kube-scheduler and controller-manager will be unavailiable. Your processes and workloads will continue during this period however, so you will not induce downtime.

Once the master node(s) come back up, you can begin updating the worker nodes. But there are a few strategies to doing this. 
1. You COULD choose to upgrade them all together, but this would cause downtime and your users could not access the applications. 
2. You could also do a "rolling update" and upgrade the nodes 1 at a time. Do this by draining a node, then proceed to upgrade that drained node, and so on.
3. You could also choose to just add new nodes to the cluster that are running a newer version. This is especially convienient on cloud provided clusters, as you can easily provision new nodes quickly. You then drain the old nodes, and remove them. (mark the old nodes as cordoned, then drain)

Remember, Kubelet will need to be upgraded seperately from the rest of the control components. 

Running `kubeadm upgrade plan` will detail out any availiable updates, and give you information on steps to take.

## Upgrading

### Master Nodes

```bash
apt-get upgrade -y kubeadm=<version>\
kubeadm upgrade apply <version>
```

Check your work with `kubectl get nodes` to see versions. You will notice that the version number did not change. This is because we need to upgrade the Kubelets on each node manually. 

To upgrade kubelet, you will run `apt-get upgrade -y kubelet=<version>` first on the master node. Then run `systemctl restart kubelet` to apply.

`kubectl get nodes` will now show the master as a higher version.

### Worker Nodes

First, `kubectl drain node01`

Then (in ssh) `apt-get upgrade -y kubeadm=<version>`\
`apt-get upgrade -y kubelet=<version>`\
`kubeadm upgrade node config --kubelet-version <version>`\
`systemctl restart kubelet`

Once the node is ready again, make sure you run `kubectl uncordon node01` to mark it schedulable again.

Perform the same steps on all worker nodes in the cluster.

# Backup & Restore Methods

## Backup Candidates

Things that you can backup are:

- resource configs
- etcd cluster
- persistent volumes

### Resource Configs

If you want to save your configurations, the preferred approach is to have created or create a definition file of said resource. This allows you to backup that file however you wish.	

Often these folders and files are stored in a SourceCode Repo like GitHub where the team can collaborate on the files.

Another way to save all resource configs is to run `kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml`

### ETCD Cluster

Rather than backing up the resources directly, you could opt to backup the whole ETCD cluster, as this contains all of those configs anyway.

In the etcd.service file made upon configuration of ETCD, you will have a --data-dir entry, pointing to where  your ETCD data is stored on the node. You can backup this directory, an so backup the config for your cluster.

You can also utilize the built in snapshotting tool by running `etcdctl snapshot save <path>/<name>` to save snapshots of your etcd cluster. To restore from a snapshot, you simply stop the api-server `service kube-apiserver stop`, then `etcdctl snapshot restore <path>/<name>`.  

**note** that when you restore from a snapthot, ETCD creates a new cluster and adds members to that cluster, and creates a new data dir. you will then need to update the etcd.service file to match the correct data dir. Then `systemctl daemon-reload` and `service etcd restart`, and `service kube-apiserver start`

## Working with ETCDCTL


`etcdctl` is a command line client for etcd.

In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.

You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:

`export ETCDCTL_API=3`
On the Master Node.

To see all the options for a specific sub-command, make use of the -h or --help flag.

For example, if you want to take a snapshot of etcd, use:

`etcdctl snapshot save -h` and keep a note of the mandatory global options.

Since our ETCD database is TLS-Enabled, the following options are mandatory:

`--cacert`          verify certificates of TLS-enabled secure servers using this CA bundle

`--cert`            identify secure client using this TLS certificate file

`--endpoints=[127.0.0.1:2379]`          This is the default as ETCD is running on master node and exposed on localhost 2379.

`--key`       identify secure client using this TLS key file


Similarly use the help option for snapshot restore to see all available options for restoring the backup.

`etcdctl snapshot restore -h`

# Certification Exam Tip!

Here's a quick tip. In the exam, you won't know if what you did is correct or not as in the practice tests in this course. You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the the kubectl describe pod command to verify the pod is created with the correct name and correct image.

# References

[https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

[https://github.com/etcd-io/website/blob/main/content/en/docs/v3.5/op-guide/recovery.md](https://github.com/etcd-io/website/blob/main/content/en/docs/v3.5/op-guide/recovery.md)

[https://www.youtube.com/watch?v=qRPNuT080Hk](https://www.youtube.com/watch?v=qRPNuT080Hk)