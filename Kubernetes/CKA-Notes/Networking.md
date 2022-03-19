---
title: Networking
description: 
published: true
date: 2022-03-19T15:57:42.393Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:57:40.250Z
---

# POD Networking Concepts

## Requirements for POD Networking

- Every pod should have it's own IP address
- Every POD should be able to communicate with every other POD in the same node.
- Every POD should be able to communicate with every other POD in other nodes without NAT.

For Understanding sake, we will do this manually rather than installing a CNI plugin first.

## Manual Network Config

1. Create and bring up a bridge network on each node
2. Each subnet will be set do a different subnet
	1. assign IP's for the node on the bridge network
3. Attach the POD to the Bridge network (need interfaces on either end of the veth) The PODs should now be able to communicate within a Node
4. To connect the PODs between Nodes, you would create a route to the other Nodes so that you can reach them. A better way however would be to set up an external router with the routes already configured.

To meet CNI reqs and automate this process, make a script to do the tasks, then group them into "ADD" and "DEL" sections. You can direct kublet to this CNI script in the config file for each kubelet.

# CNI in Kubernetes

If you look in the config file for the kubelet.service, you will see the settings for the CNI. It will tell you where your conf and config locations for your plugin are.

By default you can see the avaliable plugins at `/opt/cni/bin`

## WeaveWorks

This is for the Weave CNI plugin. If we can understand this, we should be able to translate well to others like flannel or calico.

Weave first installs an agent POD on each node in the cluster. Each agent or peer stores the config data for the cluster. Weave then creates its own bridge network between nodes.

When a POD tries to communicate with another, Weave will intercept this to see if it's on a different node. If it is, it will encapsulate the packet with the node address, then send it along to that node, where the weave agent there will then decapsulate and forward to the POD.

### Deploy Weave

Weave is deployed as a deamon or service, OR as a POD on a preinstalled K8s cluster.

`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version-$(kubectl version | base64 | tr-d '\n)"`

This method is great if you created your cluster using Kubeadm.

# IPAM (IP Address Management)

This Addresses how bridge networks and pods in a cluster are assigned IP addresses, and how those are maintained.

The CNI Plugin is responsible for assigning and maintaining IP Addresses. To manage them manually, you could keep them in a file and note them as "assigned" and "free" for example.

You can also invoke a plugin called `host_local` to handle this for you. Adding a line to your script like `ip = get_free_ip_from_host_local()`

Weave can handle it slightly different. By Default it assigns within 10.32.0.0/12. Weave will split this range between the different Nodes, so they each have equal free IPs. 

# Service Networking

- Identify Node Network Range
	- `kubectl get nodes -o wide`
	- `ip a` Look at the external interface for the node for the IP range
- Identify IP Range for PODs
	- `kubectl -n kube-system get pods`
	- `kubectl -n kube-system logs <pod-name> -c <cont name>`
		- Search for IP location range at the top of Logs
	- Can also look in the conf file for weave
- Identify IP Range for Services in the Cluster
	- `cd /etc/kubernetes/manifests`
	- `cat kube-apiserver.yaml | grep service`
- What type of proxy is kube-proxy using
	- `kubectl -n kube-system logs <kube proxy pod>`

# Cluster DNS

Remember that DNS names for your NODES will be handled by an external DNS server within your environment. This does not pertain to inside the cluster.

This is where kube-DNS comes in. This will handle internal DNS records for the cluster. It ONLY handles PODs and Services.

Lets say we have 2 pods, and a service. Test, Web, and Web-service.

Anytime a service is created, a new DNS record is created to link the service name to its assigned IP address. This means that any POD can reach the service by its DNS name. This is assuming we're in the same namespace however.

If we were in different namespaces, you would need to specify `.namespace` after the DNS name. The kube-DNS keeps track of the namespace that each service is in, as well as the fact that it is a service, and the cluster's "root domain". This means the full path to the service would be `http://web-service.apps.svc.cluster.local` assuming your service is in the apps namespace.

PODs also get a DNS record, except it replaces the "." in the IP with a "-", creating a FQDN of `http://10-244-2-5.apps.pod.cluster.local`

# CoreDNS in Kubernetes

The recommended DNS server in K8s as of 1.12 is CoreDNS. CoreDNS is deployed as a ReplicaSet in a deployment in the cluster. This runs the same executable as running it manually.

Located at `/etc/coredns/Corefile` you'll find a configuration file for this CoreDNS POD. This file will contain plugins for the DNS server, including the "Kubernetes" plugin that handles the tie-in for coreDNS to k8s. This is also where your top level "root domain" for the cluster is set.

**note** that the Corefile is passed to the pod as a configmap in `-n kube-system`

When CoreDNS is deployed, it also deploys a service called `kube-dns` as a clusterIP, so that all of the pods in the cluster can reach it. Kubelet will automatically provide the nameserver address to the PODs as they are created. This is set in the kubelet config file at `/etc/var/lib/kubelet/config.yaml`

# Ingress

What is the difference between a "service" and "ingress"?

Lets start with a scenario. You have an online store, at a given address (my-online-store.com). You deploy this website as a POD in a deployment. It also needs a DB, so you create a MySQL POD as well, and a service of type ClusterIP for the DB to make it accessible to the webapp. Now that your app i working, you need to make it accessible to the outside world.

You start by making a serivce of type NodePort, and exposing it on 38080 on the node. Now your app is accessible at "http://nodeIP:38080".
This works, but we want to be able to use DNS, because publishing the IP of the node is not realisitic. So now you create an entry in your DNS server to point to  your nodes with the url "http://my-online-store.com:38080".

This is better, but we don't want the users to need to type in the port number either. So now we bring in a proxy server to stand between the end user and your server. This allows the user to type a address like "my-online-store.com", and this directs to port 80 on your proxy, which translates to your cluster's IP & port. You would of course need to point your DNS record to this proxy server.

This all works fine if you're on a local "on-prem" cluster, but what about in the cloud? Well if you were using a hosted cluster, you can instead create a service of type LoadBalancer for you app, on port 38080. This will tell K8s to do all of the same things as a NodePort, but it will also send a request to the cloud host to configure a network LoadBalancer for this service. It creates a LoadBalancer that acts very similarly to a proxy-server on a self-hosted environment. 

Ok now what happens if we have multiple apps in the cluster? Well we can go through the exact same process as before, creating the LoadBalancer service which then triggers the creation of a host loadbalancer for this service.\
This would work fine, but you also will be paying for each loadbalancer you deploy, so this might not be the most cost effective way to do things.

Well, you could go ahead and create yet another layer of LoadBalancing, so that you have a load balancer that is then directing requests to a given loadbalancer below it that is specific to the service.

As you can see, this is a pretty complex configuration that would require a significant amount of management and cost. So what if we could do all of this INSIDE k8s, as another object with a definition file? This is what Ingress is. Ingress provides a singular externally accessible URL that you can config to route to different services based on the URL path, and include SSL certs. You can think of this very similarly to a Layer 7 LoadBalancer. Remember though that you still need to publish your ingress service externally, but you only have to do this once. 

## Working with Ingress

Remember that you will need to configure this yourself. K8s does not come default with an Ingress Controller deployed. There are multiple to choose from like NGINX, HAPROXY, and Traefik. You will also need to create ingress resources for the controller to utlize once it is deployed.

Ok so how do we deploy an Ingress Controller? We're going to focus on NGINX for this example.

To start, this is just deployed as any other deployment in K8s. Start with a def file like:
```yaml
apiVersion: extensions/v1beta1
kind: apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ing
  template:
    metadata:
      labels:
        editor: nginx-ingress
    spec:
      containers:
      - name: nginx-ingress-controller
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller
      args:
        - /nginx-ingress-controller
      env:
        - name: name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: namespace
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```
**NOTE** it would be helpful to create a configmap for nginx settings down the line.

Now we will need a service to expose this to the external world. Create a NodePort for the deployment:
```yaml
apiVersion: v1
kind: apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  selector:
    app: nginx-ingress
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
```

You will also need to create a Service Account to be able to utilize the ingress controller.

### Ingress Resources

Now that we have an Ingress Controller, we need to create Ingress Resources to direct the Controller on what to do with traffic it receives.

Here is an example resource file to direct traffic to the "wear-service" in an example cluster.
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
	  servicePort: 80
```

This simple resource will direct ALL traffic to the wear-service "service" in your cluster.

"Rules" are used if we want to route traffic based on different parameters like subdomains or subpages. These are called Paths.

So how do we create these rules? here is an expanded def file:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
```

**note** when you create the resource, you might notice it has a "default serivce" configured. This is to be used anytime a user tries to access a page that does not exist. You will need to configure this service and point it somewhere.

Here is a side-by-side of two resource def files, one using subpages, and one using subdomains:
![23237114b160d8a72003f932a692ef37.png](:/5ed9ab979a9749fca0caed102920e90e)

Article: Ingress

As we already discussed Ingress in our previous lecture. Here is an update.

In this article, we will see what changes have been made in previous and current versions in Ingress.

Like in apiVersion, serviceName and servicePort etc.
![1d8f63c034b2c7383765dddc8279817d.png](:/ae6e8bc920fb40e398d525e1932dc2fd)

Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-

Format - `kubectl create ingress ingress-name --rule="host/path=service:port"`

Example - `kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"`

Find more information and examples in the below reference link:-

[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-)

References:-

[https://kubernetes.io/docs/concepts/services-networking/ingress](https://kubernetes.io/docs/concepts/services-networking/ingress)

[https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types)

## Ingress - Annotations and rewrite-target

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen [here](https://kubernetes.github.io/ingress-nginx/examples/). I would like to explain one such option that we will use in our labs. The [Rewrite](https://kubernetes.github.io/ingress-nginx/examples/rewrite/) target option.

Our watch app displays the video streaming webpage at `http://<watch-service>:<port>/`

Our wear app displays the apparel webpage at `http://<wear-service>:<port>/`

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/`

Without the rewrite-target option, this is what would happen:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/watch`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/wear`

Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.


To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

For example: replace(path, rewrite-target)
In our case: replace("/path","/")

```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: test-ingress
      namespace: critical-space
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /pay
            backend:
              serviceName: pay-service
              servicePort: 8282
```

In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")
```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$2
      name: rewrite
      namespace: default
    spec:
      rules:
      - host: rewrite.bar.com
        http:
          paths:
          - backend:
              serviceName: http-svc
              servicePort: 80
            path: /something(/|$)(.*)
			```