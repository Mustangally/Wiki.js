---
title: Security
description: 
published: true
date: 2022-03-19T15:54:48.480Z
tags: 
editor: markdown
dateCreated: 2022-03-19T15:54:46.179Z
---

[Kubernetes+-CKA-+0600+-+Security.pdf](:/370da7cc597742eba0734b4953f94658)

[Networking.pdf](:/d060c9b946594a289f9155dbdce6d874)

# Security Primitives

To make a host secure, a few things should initially be true in almost all circumstances.
- Password Auth should be DISABLED
- Root Access Disabled
- SSH Key Auth ONLY for login
- Other network wide security policies in place

## Controlling Access to kube-apiserver

The first line of defense for the k8s cluster itself is the apiserver, as all communication flows through here.
A few things should be addressed:
### Who Can Access?
This is defined by the definition mechanisms like files with usernames & passwords, usernames & tokens, certificates, or external auth like LDAP.
For Machines, you would create service accounts to access

### Permissions

Auth mechanisms are used to define roles for users who have access to the cluster. These roles define what they can and can't see or do.
Some mechanisms are RBAC Auth, ABAC Auth, Node Auth, Or Webhook Auth.

## TLS (Transport Layer Security) Certificates

All of the communication between different K8s components are done with TLS certificates. This is to ensure that communication is encrypted and secure.

##  Inter-App Communications

By Default, All PODs can access all of the other PODs in the cluster, but this can be restricted.

# Authentication

4 General classes of users that you may have accessing your cluster are (not really restricted to):
- Admins
- Developers
- End-Users
- Bots/Service Accounts

K8s does not really handle user account creation or management for human class accounts. This would mean that admins and devs do not have their accounts stored in K8s configuration. HOWEVER, bots/serviceaccounts ARE managed by K8s, and can be created with `kubectl create serviceaccount sa1`, `kubectl get serviceaccount`

All user access is handled by the kube-apiserver. This is true whether you access through `kubectl` or the cluster's direct address. They all go through the apiserver.
Before you can get THROUGH the apiserver, the kube-apiserver must authenticate the request before passing it along.

To do this, there are a couple different mechanisms that can be used.
- Username/Password auth file stored on the cluster
- Username/Token file
- Certificates
- 3rd parth auth protocols.

### Static Files

**NOTE** This is not the recommended auth mechanism in K8s, as it is not particularly secure.

You can create a list of users and passwords, store them in a csv, and use these for authentication. You must have 3 columns in this file. Password, username, and userID. (in that order). You can also add a 4th "group" column to assign groups to a user.

You would then pass that file through as an option to the apiserver in the kube-apiserver.service file (`--basic-auth-file=<name>.csv`). If you set up your cluster with `kubeadm`, you must update the kube-apiserver definition file to include this option. (Located in the manifests directory of the master node(s))

You could optionally also use a `--token-auth-file` to use tokens reather than passwords. This is still a .csv file.

Both of these methods are used if you're accessing the cluster via the cluster's direct address. You would access it as so:\
`curl -v -k https://maste-node-ip:6443/api/v1/pods --header "authorization: bearer <token>"` or\
`curl -v -k https://maste-node-ip:6443/api/v1/pods -u "user1:password123"`

### Article on Setting up Basic Authentication

Setup basic authentication on Kubernetes (Deprecated in 1.19)
Note: This is not recommended in a production environment. This is only for learning purposes. Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases

Follow the below instructions to configure basic authentication in a kubeadm setup.

Create a file with user details locally at /tmp/users/user-details.csv

#### User File Contents
```
password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
```
Edit the kube-apiserver static pod configured by kubeadm to pass in the user details. The file is located at `/etc/kubernetes/manifests/kube-apiserver.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```

Modify the kube-apiserver startup options to include the basic-auth file


```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
      <content-hidden>
    - --basic-auth-file=/tmp/users/user-details.csv
```
Create the necessary roles and role bindings for these users:

```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
 
---
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Once created, you may authenticate into the kube-api server using the users credentials

`curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"`

# TLS Certificates Basics (Pre-Req)

The basic purpose of a TLS Cert is to guaruntee trust between two parties. For example, when a user tries to access a website, at TLS Cert is what ensures that connection is encrypted and the two parties are genuine.

## Symmetric Encryption

The basic process of symmetric encryption is this: The sender encryptes the data using a key. They then send the data along with the key to the endpoint, so that the endpoint can then decrypt that data to read it.

The trouble here, is a man-in-the-middle can grab both the data and the key, and so read the data.

## Asymmetric Encryption

This method uses a pair of keys to do the encryption. A private and public key. For the sake of simplicity, think of the public key as a "lock", making this a key & lock pair.

The private key should ALWAYS stay with the owner, or sender of the data. The Public key (lock) is shared between the two parties. The private key is what is used to decrypt the Public Key.

The public key is used to encrypt data, but it CANNOT decrypt data. The private key is what decrypts the data. So in a two way exchange, you would swap public keys, then encrypt the data you want to send with the other parties public key. This way, only the other party who has the matching private key can decrypt and read the data.

### SSH Example

SSH comms are a good example of encrypting traffic and using keys for authorization. Often times you don't wan to to use password auth to a ssh server, but would rather use ssh-keys instead.

The client starts by generating a keypair `ssh-keygen`, then you put the public key on whatever server you need access to (`~/.ssh/authorized_keys`. Then, lock down the server so that it can only be accessed through that public key. When you try to login to the server, you would use your private key to unlock the public key that grants auth to the server. 

**NOTE** that although we are discussing encrypting with the public key and decrypting with the private, you CAN do it the other way around, as is done with SSH to securely communicate both directions. Just be aware that this means anyone with your public key can decrypt your data.

## Asymmetric to Symmetric Encryption Communications

A typical web interaction would go like this:

When connection is initiated by the client, the web server sends over its public key. The client then encrypts it's SYMMETRIC key using the public key from the server, and sends it back. This way that key is not readable by anyone but the server who has the private key matching the initial public key. The server then decrypts the symmetric key from the client, and symmetric encryption communication can now proceed.

## Certificates

Where certificates play into all of this, is to prove that they key you recieved when going to a website, is actually the key for that website. It ensures that you are not beeing fooled and redirected to a bad website that just looks and acts the same, but is not where you intend to be. It will include the public key & who that key is issued to (DNS record), so that you can be sure the key you recieved belongs to the server you wanted to communicate with.

But how do we know that the certificate wasn't faked? This is where the signature or issuer on a cert comes into play. You can sign it yourself, and this is known as a "self-signed certificate". However, this is not usually trusted as anyone can self-sign a certificate.

Instead, legit certificates will be signed by a Certificate Authority (CA). To get a CA Signed certificate, you will generate a CSR (certificate signing request) and send that to a CA to sign. They will validate the certificate, sign it, and return it to you for use. 

But how do we know the CA was legit? Well each CA has its own public and private key pairs. When they sign a cert, they use their private key to do so. The public keys for all of the CA's are automatically built into most web-browsers we use today. When you receive a cert from a website, your browser checks the cert against the public keys from the CA that supposedly signed the cert, and verifies that it is genuine.

Obviously, all of this so far has been for public internet use. But what about internal certs? How can you generate signed certs within your organization? Most of the popular CAs offer a private offering of their services, so you can run a CA server within your org that signs certs internally, and so you can sercurly browse intranet traffic. You would of course need to add the public key of your CA into all of the browsers for your users.


### Client Validation

So far we have figured out how the client determines the legitimacy of the server. But how does the server confirm the client?

The server can request that the client generate a csr, have it signed, and send it over to confirm identity.

This however is not common on web-servers, which you may have noticed considering you've never made one for yourself as a client, although this may have been done "under the hood" for you from time to time.

# TLS in K8s

Now that we have an understanding of TLS, how do we use this in K8s to secure communication in, to, and from your cluster?

We will walk through the different components and discuss how they interact:

### Server Components

**kube-apiserver** will generate a crt and a key, that it will then use to communicate. For this guide, lets call it apiserver.crt & apiserver.key

**Etcd** will also generate it's own keypair, etcd.crt & etcd.key

**kubelet** will generate kubelet.crt & kublet.key

### Client Components

**Admin Users** require their own keypair, admin.crt & admin.key

**kube-scheduler** will also generate their own pair as it communicates with the apiserver to perform scheduling

**kube-controller-manager** also requires a keypair for apiserver comms

**Kube-Proxy** will generate a keypair as well

### CA in K8s

K8s requires that you have at least 1 CA sign the certificates for use within the cluster.

## Generate Certificates

For this guide, we will use OPENSSL to generate the certs needed in K8s.

First, create a private key for the CA by running `openssl genrsa -out ca.key 2048`\
Then, create a CSR by running `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`\
Then you will sign the cert by running `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`\
**Note** that you just technically self-signed your CA cert.

**Admin User** - `openssl genrsa -out admin.key 2048` - `openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr` - `openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`\
**Note** that you should add the group "system:masters" to your admin.crt's so that they have the proper permissions to the cluster. you can do this by adding `/O=system:masters` to the `-subj` line of your csr. This will ensure you have admin privilages when you authenticate to the cluster.

### System Component Naming

Please note that when you generate certs for the system components (scheduler, controller-manager, & proxy), you need to preface the CN name with "system" (`"/CN=system:kube-scheduler"`)

### Server-Side Components

Note that if you are in an HA environment, you will need to generate certs for each replica of a service like etcd, so that the comms between the different replicas is secure. These are called "Peer Certificates"

### Kube-apiserver

Remember that the apiserver is the central point of comms, and will go by many different names depending on who is contacting it. These different names are "aliases". The other names it may go by are:
- kubernetes
- kubernetes.default
- kubernetes.default.svc
- kubernetes.default.svc.cluster.local
- ``<node IP>`
- `<Internal IP>`

In the CSR, you need to specify the alternate names in a config file. This will be a `openssl.cnf` file, that looks like this:
```
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = <nodeIP>
IP.2 = <InternalIP>
```

You will then pass this into the csr command as so: `openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf`

## How to View Certificates

There are a few ways to inspect the certs on your cluster.

This depends on how the cluster was built. If it was done "The Hard Way", then the certs were generated manually and stored, and you can reference `/etc/systemd/system/kube-api-server.service` for example. If they were done with kubeadm, They were auto generated and stored and you can reference `/etc/kubernetes/manifests/kube-apiserver.yml` for example.

For this guide, we'll assume Kubeadm was used. You can use the attached excel sheet as reference.\
[kubernetes-certs-checker.xlsx](:/2f8e107f87114685926a5a21f807b09e)

In a cluster built with kubeadm, start in the `/etc/kubernetes/manifests/kube-apiserver.yml` file to view server components.\
You can decrypt a cert and inspect it by running `openssl x509 -in <path> -text -noout`. This will print all info for the certificate.

IF BUILT MANUALLY\
If you begin running into errors, try checking the system logs, like `journalctl -u etcd.service -l` (for etcd logs)

IF BUILT WITH KUBEADM\
try `kubectl logs etcd-master` to again see the etcd logs.

If some of the k8s functions are down, you'll need to use docker to inspect logs. You can do this by running `docker ps -a` to view container ID's, then `docker logs <ID>` to view logs.

# Certificates API

The "CA Server" for the kubernetes cluster is really just the keypair that is used to sign certificates for the rest of the cluster. So really, wherever your CA certs lie, this is you CA Server. Anytime you want to sign a cert, you need to login to that server to do so.

Often times, the master node is the CA server for the cluster. Kubeadm will set up the server this way.

The K8s Certificates API is a tool within the cluster that handles signing requests automatically, so you don't have to login and manually do this each time. You instead generate an object called a `certificateSigningRequest` object, and send this to your certificate API. These requests can be viewed and approved via the kubectl commands.

you would go about the process like this:

`openssl genrsa -out jane.key 2048`\
`openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`\
Encode the csr with base64, `cat jane.csr | base64`, then create a signing request object with a manifests definition file:

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
   - system:authenticated
  usages:
   - digital signature
   - key encipherment
   - server auth
  reques:
    <encoded text>
```

You can review all signing requests by running `kubectl get csr`, then approve with `kubectl certificate approve jane`. You can view the Cert by running `kubectl get csr jane -o yaml`. Remember that the cert will be encoded with base64, so you'll need to decode it with `echo "<text>" | base64 --decode`.

All of the CA Signing operations are handled by the Controller-Manager component of the control plane. Inside of this, you will find `CSR-APPROVING` & `CSR-SIGNING` controllers, which are responsible for carrying out these tasks. Inside of the Controller-Manager manifest file, you will find options passed in declaring the location of the CA crt and key files.

# KubeConfig

Now that we have our certs and can authenticate to the server, how do we integrate this with the `kubectl` command? One way (the manual way) is to pass in the auth parameters with the kubectl command:
```kubectl get pods 
     --server my-kube-playground:6443
	   --client-key admin.key
	   --client-certificate admin.crt
	   --certificate-authority ca.crt
```

But this way is a bit long and tedious. So, you can instead move these to the "kubeconfig" file (normally in your user home directory, under '.kube'). If you place it here properly, you don't have to specify the config file that you're using when you run a `kubectl` command.

## KubeConfig Structure

The kubeconfig file has 3 different sections. Clusters, users, and contexts.\
- Clusters are all of the different clusters you will be accessing (prod, dev, google, etc)
- Users are obviously the user accounts you use to access the clusters (admin, dev user, prod user, etc)
- Contexts are what join these two groups together. (admin@prod)

```yaml
apiVersion: v1
kind: Config
clusters:
  - name: my-kube-playground
    cluster:
	     certificate-authority: ca.crt
		   server: https://my-kube-playground:6443
contexts:
  - name: my-kube-admin@my-kube-playground
    context:
	    cluster: my-kube-playground
		  user: my-kube-admin
users:
  - name: my-kube-admin
    user:
	    client-certificate: admin.crt
		  client-key: admin.key
```
You can add as many clusters, contexts, and users as you need. You can also optionally add a `namespace` field to your contexts, to set a default namespace for that context.

#### Context

But now how does `kubectl` know which context to use?

One way is to add `current-context:` to the top of your file after `kind`

The other way is to run `kubectl config use-context <context>`

You can look at your context and config with `kubectl config view`

# Persistent Key/Value Store
We have already seen how to secure the ETCD key/value store using TLS certificates in the previous videos. Towards the end of this course, when we setup an actual cluster from scratch we will see this in action.

# API Groups

The focus of this lecture are the `/version` & `/api` paths of your cluster (`https://my-kubernetes-cluster:6443/version`)

The Kubernetes API is grouped into different categories related to their function. For example, you have groups like /metrics, /healthz, /apis, /logs, etc.

We are going to focus on `/api` & `/apis`.

- /API is the "Core Group", or Where all the core functionality is grouped
	- namespaces
	- pods
	- events
	- endpoints
	- nodes
	- etc
- /apis is the "Named Group", and where all newer features will be grouped.
	- apps
		- /v1
			- deployments
			- replicasets
			- statefulsets
	- extensions
	- networking
		- v1
			- networkpolicies
	- storage
	- auth
	- certs
		- v1
			- csr's

Each of the resources will have a list of actions or "Verbs" associated, things like:
- get
- list
- create
- delete
- etc

## KubeProxy

**SideNote** if you want to access the cluster on localhost, you can run `kubectl proxy` to open a proxy using your credentials into the server. This will use port 8001 on localhost.

`Kube proxy` and `Kubectl proxy` are NOT the same. Remember that we disussed kube proxy before, and it is how PODs communicate with eachother within the cluster, and will be discussed more in Networking

# Authorization

Why do we need authorization? Well authorization is what restricts or allows different users to perform different functions, or access certain aspects of the cluster. For example, you may want to allow Devs to view the cluster but not modify it.

## Node Authorizer

Node Authorizer is what grants different access to the components of the cluster. For example, note that when we gave the kubelet a cert, we prefaced the name with system:node. That system:node group is what defines the permissions for kublet, and anything else in that group.

## ABAC

Attribute Based Authorization is where you associate a user or group with a set of permissions. This is done by creating a Policy File, with a set of policies defined in a JSON format. **Note** that this is hard to manage as changes must be manually made to the file whenever changes are needed. For this reason, it is not a preffered method of authorization.

## RBAC

Role Based Authentication is much easier. Rather than associating permission to a user or group, we would instead create a "role" and assign permissions to it. You then just associate the user to the role, and manage it through the one point.

## Webhook

A webhook authorization is how you allow a 3rd party tool to manage authroization. 

## Always Allow & Always Deny

The "Always Allow" option is the default authorization mechanism in Kubernetes. you can however define the different modes you want to use in the api-server manifest file.

## Working Together

You can configure mulitple modes at once. For example, lets set Node, RBAC, and Webhook modes.

Your request to the api server will be authorized in the order that is defined in the manifest file. In this case, node > rbac > webhook.

If a user in this case sent a request, it would first hit the Node Authorizer, which ONLY handles node requests so it denys the request. It then gets passed to RBAC, which recognizes the user and approves. **Note** that anytime a req is denied, it will be passed on to the next authorizer. Once approved, no more checks are needed.

# RBAC in Detail

How do we create a role? Well, it is done with a definition file like usual.
```yaml
apiVersion: rbac.authorization.k8s/v1 #**NOTE THE API GROUP AS DISCUSSED EARLIER**
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
	  verbs: ["list", "get", "create", "update", "delete"]
```
`kubectl create role -f <file>`

Now we need to link a user to the Role. for this, we need to create a "role binding" object. Again, this is a yaml definition file:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: roleBinding
metadata:
  name: devuser-developer-binding
subjects:
  - kind: User
    name: dev-user
	  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```
`kubectl create -f <Name>`

`kubectl get roles`, and `kubectl get rolebindings`

Now how do we check access?

you can run `kubectl auth can-i create deployments` to ask the api-server if you would be allowed to create deployments. Any command can be put in here, and you will be told yes or no.

You can also check another user with `kubectl auth can-i create deployments --as dev-user`

# Cluster Roles & Cluster Role Bindings

Remember that roles & role bindings are specific to a namespaces. A Cluster Role and Cluster Role Binding are agnostic to namespaces.

These are used to allow roles to manage cluster scoped resources like nodes and PV's. You can also use them to allow users to access namespaced resources across all namespaces.

The definition syntax is the same as a role, but with the `kind: clusterRole`

# Service Accounts

As opposed to user accounts, service accounts are meant for machines or bots. Things like monitoring service like Prometheus of CI/CD like Jenkins would have service accounts.

To create a service account, run `kubectl create serviceaccount <name>`

When this is created, it also automatically generates a token. This token is what needs to be used to authenticate by the external service.

you can find the location of the token file with the `describe` command, then try `kubectl describe secret <token>`

But what happens if your "external" application is actually hosted on your cluster? You can actually mount the token file into the pod so that it is native to the application and doesn't have to be manually entered.

**Note** that each namespace automatically gets a default SA called "namespace-sa".

If you want to specify a different service account than the default for a given namespace, you can put it in as `serviceAccountName`  in the POD spec, at the same tier as containers. Remember though that you cannot modify the SA for a running POD. Changing the SA in a deployment will trigger a new rollout for that deployment.

You can also choose not to mount a SA by adding `automountServiceAccountToken: false` in the pod spec.

# Image Security

Remember that K8s images use the same format as Docker images. This means that you always have registry/user/image/tag formatting. The default reg and user are docker.io/library, and this is why you can deploy nginx by just typing "nginx". But if you wanted to pull an image made by you or your team, you may need to change these fields.

You can choose to create a private repository to keep your images in so that others cannot use them. To specify this in a pod definition file, you would make sure to add registry/user/image in the image field.

But to ensure you can authenticate to your registry, you need to create a secret file with the reg creds inside. You would run something like:
```bash
kubectl create secret docker-registry regcred \
  --docker-server= private-registry.io \
  --docker-username= reg-user \
  --docker-password= reg-passwd \
  --docker-email= youremail
```

Then inside of the pod definition file, you would add `imagePullSecrets` under the POD Spec.

# Security Contexts

This is very similar to adding the user/group in docker/docker-compose. You are defining the user that the POD or container is running as.

under Spec, you add a `securityContext` option, and a `runAsUser:` option below that.

If you want to add a capability to the container (THIS CANNOT BE DONE AT THE POD LEVEL), add:
```yaml
capabilities:
  add: ["capability"]
```

# Network Policies

Think of this like firewall rules within your cluster. By default, all PODs can talk to eachother. But sometimes we need to restrict this. To do so, we set up ingress & egress rules, and match them to labels & selectors.

Network Policy Def File Example:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
	    role: db
  policyTypes: #one or both
  - Ingress
  - Egress
  ingress:
  - from: #the dashes act as an "OR" qualifier, othewise they act as "AND"
    - podSelector: #confine to set of PODs
	      matchLabels:
		      name: api-pod
		  namespaceSelector: #confine to Namespace
			  matchLabels:
			    name: prod #label must be set on namespace
		- ipBlock: #allow an IP Address
		    cidr: 10.0.0.1/24
	  ports:
	  - protocol: TCP
	    port: 3306
		
# lets look at egress too
  - egress:
    - to:
	    - ipBlock:
		    cidr: 10.0.0.10/24	
	    ports:
		   - protocol: TCP
		     port: 80
```

**NOTE** Not all network solutions support Network Policies, but refer to docs to see if this is updated. As of the course, Flannel does not support the policies.