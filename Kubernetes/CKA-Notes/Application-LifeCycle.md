---
title: Application LifeCycle
description: 
published: true
date: 2022-03-19T15:51:45.755Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:51:43.590Z
---

[Kubernetes+-CKA-+0400+-+Application+Lifecycle+Management.pdf](:/18e7c69bfd3f455e9d9bcc53204c21fd)

# Rolling Updates & Rollbacks

When you first create a deployment, it kicks off a rollout. When you go through an update, it triggers a second rollout. In this process, it will sequencially kill and re-start a POD with the new config.

`kubectl rollout status <deployment-name>`

`kubectl rollout history <deployment-name`

`kubectl apply -f <deployment-name.yml`

`kubectl set image deployment/myapp-deployment \
  nginx=nginx:1.9.1`


## Deployment Strategies

- Recreate - Kill all then recreate
- Rolling Update (DEFAULT)

## Upgrades

When a new deployment is created, a ReplicaSet is created to fulfill the correct number of PODs.

An Upgrade, creates a new ReplicaSet and rolls out a new set of PODs to replace the old ones. 

`kubectl rollout undo <deployment-name>`

This will do a "reverse rollout" to redeploy the originial config.

# Configure Applications

## Commands and Arguments

If you need to pass in an argument to your container, you would add the `args: ["stuff"]` parameter in your yaml, at the same level as container name and image. This is the same as a `CMD` in a dockerfile, or a command in a `docker run` statement.

The `command:` field can also be used in a k8s yaml, and this would be much like the `ENTRYPOINT` in dockerfile. This can often be a prefix to the `args:` when the container runs.

## Environment Variables

You can apply `env` array (items start with -) along the same line as a container name or label.

This can be used to pass things like key-values, configmaps, and secrets to the container.

### valueFrom

`valueFrom` goes under a env variable to point the env to a file, like a secrets file. 
```yaml
env:
  -name: app
   valueFrom:
     secretKeyRef:
```

## Config Maps

Rather than having to handle a large number of `env` variables, you can utilize a ConfigMap to store them all for you, then just pass the one variable into the container.

You would take the key-value-pairs and place them in the ConfigMap (`app-name: app`).

you would then modify the `env` line in you definition file as so:
```yaml
envFrom:
  - configMapRef:
      name: app-configmap
```

To create a configmap, you can go one of two ways.

### Imperative: 

`kubectl create configmap`

```bash
kubectl create configmap <config-name> --from-literal=<key>=<value>

kubectl create configmap app-configmap --from-literal=APP_COLOR=blue
```
To add more lines to the configmap, just repeat the `--from-literal=` line again. This can get a bit messy when lots of variables are involved.

You can also create a configmap from a file already containing all of the data. to do this, run\
`kubectl create configmap <config-name> --from-file=<file-path>`

### Declarative: 

`kubectl create -f`

First you need to create a definition.yml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app_color: blue
  app_mode: prod
```
`kubectl create -f config-map.yml`

### View ConfigMaps

`kubectl get configmaps`

`kubectl describe configmaps`


### Apply to Container

Under the container, add:
```yaml
envFrom:
  -configMapRef:
     name: app-config
```
The configmap named in the deployment will reference a deployed configmap on the system.

You can also choose to only use one variable from the configmap, by defining the  `key` underneath the name of the configmap you're referencing.

```yaml
envFrom:
  -configMapRef:
     name: app-config
	   key: APP_COLOR
```

## Secrets

Rather than hardcoding the username and passwords into a container, you can use Secrets to store any sensitive data for a container.

These are usually stored in an encoded or hashed format.

### Imperative:

`kubectl create secret generic`

`kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-litreal=DB_PASS=Pass`

Again, this can get complicated if there are too many entries.
SO, you can again use a file to create your secrets file:\
`kubectl create secret generic app-secret --from-file=app_secret.properties`

The data in this file should be stored in key-value-pair format.

### Declarative:

`kubectl create -f .yml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_PASS: passwd
```
`kubectl create -f secret-data.yml`

#### HOWEVER

You need to store that data in a Hashed format. To convert your data into hashed strings, you can run `echo -n 'mysql | base64'` to hash the string `mysql`

### View Secrets

`kubectl get secrets` & `kubectl describe secrets`

To see the values in the file, out put to a yaml with `-o yaml`

### Inject into a POD

```yaml
envFrom:
  - secretRef:
      name: app-secret
```
`kubectl create -f pod.yml`

## A note about Secrets!

Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret) page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it. 

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

- Not checking-in secret object definition files to source code repositories.
- [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in ETCD.  

Also the way kubernetes handles secrets. Such as:

- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the protections and risks of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks)

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault.

# Multi-container PODs Design Patterns

There are 3 common patterns, when it comes to designing multi-container PODs. The first and what we just saw with the logging service example is known as a side car pattern. The others are the adapter and the ambassador pattern.

![4c663ab0827622e03c119ddf627f8edd.png](:/ca32a70648f040d983112cd2f9261d77)

# InitContainers

In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.


But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where initContainers comes in.


An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:

``` yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

[https://kubernetes.io/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

# Self Healing Applications

Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.