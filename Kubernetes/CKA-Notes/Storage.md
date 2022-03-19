---
title: Storage
description: 
published: true
date: 2022-03-19T15:55:48.894Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:55:46.912Z
---

[Kubernetes+-CKA-+0700+-+Storage.pdf](:/34ff803cc8af44638dbd2a372b138327)

# Into to Docker Storage

To start, when you install docker, a dir under `/var/lib/docker` is created to contain most docker data. Under here, you will have things like `containers`, `images`, `volumes`, etc.

## Layered Architecture

When docker builds images, it creates them in layers, adding changes onto the first. You would typically have the `base layer`, `apt changes`, `pip changes`, `source code`, and then `update for the entrypoint`.

Docker also will store these layers in its own file structure, so that when you create another dockerfile image, it will re-use the data from other images that use the same resources. This means that you don't need to store the ubuntu base layer twice for example.

Once you run a `docker build` command, it packages all of these layers together in a read-only file. You cannot change the components after it is built. When you actually RUN a container, it adds a 6th layer on top of the image, that is the read/write layer and is used to store data created by the container. (data, logs, etc.) The life of this layer however is only as persistant as the container. This is the "container layer".

If you want to make a change to something in the image layer while the container is running, docker will create a new copy of that file in the container layer, and this copy will be read/write. Remember though, that any changes here will only last as long as the container, and will not follow the image.

### Volumes

So what happens if we want to persist data for a container?

This is typically done with a `docker volume create` command. This creates a new directory under the `/var/lib/docker/volumes/` directory. Then, when you run the container, you add a `-v <volume>` flag and mount it to a location in the container. This mounts the volume to the read/write layer of the container.

**note** that docker can auto create the volume if it is specified in the run command, but not created ahead of time.

Optionally, you can also mount a volume that is not in the default docker volumes directory, by simply specifying the path to the volume rather than a volume name. This is called a "bind mount" rather than a "volume mount".

# Container Storage Interface (CSI)

With CSI, the intent was to open compatability to different storage drivers and techs so that you can utilize any storage interface that you want. Rather than being set on one storage solution, you can integrate whatever you need.

# Volumes

Remember that volumes are used to retain data from a container, so that it is not destroyed when the container or POD is destroyed.

In a def file in kubernetes, you define the volume at the same level as containers as so:
```yaml
volumes:
  - name: data-volume
    hostpath:
      path: /data
	    type: Directory
```

You would then apply it to a container by adding this to the container's definition:
```yaml
volumeMounts:
  - mountPath: /opt
    name: data-volume
```

Notice that we mounted the volume to a path on the Node. This would obviously work fine if you only have 1 Node. But what happens when you migrate to another Node the next time the POD is run?

You could connect it to a network file location via NFS or similar. There are many options here, and it depends on what platform you're using.

# Persistent Volumes (PV)

One problem with the traditional approach to volumes is that since it is defined at the POD level, you would have to define this for every POD or Deployment that you make. But when you have a large environment, this can be come cumbersome and hard to manage. Rather than this, we can use Persistent Volumes to manage this all more centrally.

A Persistent Volume is a cluster wide pool of storage, that users or PODs can use PVC (persistent volume claims) to carve out pieces as needed.

A PV definition file looks like this:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
	capacity:
	  storage: 1GB
	hostPath:
	  path: /tmp/data
```
`kubectl create -f <file>`

You can also change `hostPath` to something different if you're using some kind of networked storage. Check the documention as required.

# Persistent Volume Claims (PVC)

PVC's are how a user or POD attaches PV to a POD. **note** that there is a 1-to-1 relationship between PV's and PVC's. This means you cannot have multiple claims to a PV.

a PVC def file looks like this:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - readWriteOnce
	resources:
	  requests:
	    storage: 500MB
```

## Using PVCs in PODs

Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:

```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      containers:
        - name: myfrontend
          image: nginx
          volumeMounts:
          - mountPath: "/var/www/html"
            name: mypd
      volumes:
        - name: mypd
          persistentVolumeClaim:
            claimName: myclaim
```

The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.

Reference URL: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

# Additional Topics

We discussed how to configure an application to use a volume in the "Volumes" lecture using volumeMounts. This along with the practice test should be sufficient for the exam.

Additional topics such as StatefulSets are out of scope for the exam. However, if you wish to learn them, they are covered in the  Certified Kubernetes Application Developer (CKAD) course.

# Storage Classes

## Static Provisioning

This is when you pre-define the storage block that your volume lives on.

## Dynamic Provisioning

This is where "storage classes" come in. you create a storage class in K8s that ties to something like GCE. This allows for the underlying storage to be dynamically created when a PVC is created. You would define the storageClass in the PVC is created. Add `storageClassName: example` in the `accessModes:` section of the PVC.

Again, Look at the Documentation for the different SC plugins avaliable. These can be anything from NFS to Cloud Providers.

A Storage Class definition file looks like:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none
```

You can also create different SCs that meet different requirements in terms of speed and replication, and this is why it's called storage "Classes"