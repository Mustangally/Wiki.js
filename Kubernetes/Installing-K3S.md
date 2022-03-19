---
title: Installing K3S
description: 
published: true
date: 2022-03-19T19:54:41.954Z
tags: kubernetes, k3s
editor: markdown
dateCreated: 2022-03-19T19:54:39.282Z
---

<p id="bkmrk-an-installation-of-k">An installation of Kubernetes that was half the size in terms of memory footprint.</p>
<h2 id="bkmrk-prerequisites">Prerequisites</h2>
<p id="bkmrk-these-instructions-a">These instructions assume you have set up two nodes, a load balancer, a DNS record, and an external MySQL database as described in <a href="https://rancher.com/docs/rancher/v2.6/en/installation/resources/k8s-tutorials/infrastructure-tutorials/infra-for-ha-with-external-db/">this section.</a></p>
<p id="bkmrk-rancher-needs-to-be-">Rancher needs to be installed on a supported Kubernetes version. To find out which versions of Kubernetes are supported for your Rancher version, refer to the <a href="https://rancher.com/support-maintenance-terms/">support maintenance terms.</a> To specify the K3s version, use the INSTALL_K3S_VERSION environment variable when running the K3s installation script.</p>
<h3 id="bkmrk-1.-install-kubernete">1. Install Kubernetes and Set up the K3s Server</h3>
<p id="bkmrk-when-running-the-com">When running the command to start the K3s Kubernetes API server, you will pass in an option to use the external datastore that you set up earlier.</p>
<div id="bkmrk-connect-to-one-of-th">
<div>
<ol>
<li>Connect to one of the Linux nodes that you have prepared to run the Rancher server.</li>
<li>
<p>On the Linux node, run this command to start the K3s server and connect it to the external datastore:</p>
<pre><code>curl -sfL https://get.k3s.io | sh -s - server \
--datastore-endpoint="mysql://username:password@tcp(hostname:3306)/database-name"
</code></pre>
<p>To specify the K3s version, use the INSTALL_K3S_VERSION environment variable:</p>
<div>
<pre><code class="language-sh" data-lang="sh">curl -sfL https://get.k3s.io |  INSTALL_K3S_VERSION=vX.Y.Z sh -s - server \
--datastore-endpoint="mysql://username:password@tcp(hostname:3306)/database-name"</code></pre>
</div>
<p>Note: The datastore endpoint can also be passed in using the environment variable <code>$K3S_DATASTORE_ENDPOINT</code>.</p>
</li>
<li>
<p>Repeat the same command on your second K3s server node.</p>
</li>
</ol>
</div>
</div>
<h3 id="bkmrk-2.-confirm-that-k3s-">2. Confirm that K3s is Running</h3>
<p id="bkmrk-to-confirm-that-k3s-">To confirm that K3s has been set up successfully, run the following command on either of the K3s server nodes:</p>
<div id="bkmrk-sudo-k3s-kubectl-get">
<div>
<pre><code>sudo k3s kubectl get nodes
</code></pre>
</div>
</div>
<p id="bkmrk-then-you-should-see-">Then you should see two nodes with the master role:</p>
<div id="bkmrk-ubuntu%40ip-172-31-60-">
<div>
<pre><code>ubuntu@ip-172-31-60-194:~$ sudo k3s kubectl get nodes
NAME               STATUS   ROLES    AGE    VERSION
ip-172-31-60-194   Ready    master   44m    v1.17.2+k3s1
ip-172-31-63-88    Ready    master   6m8s   v1.17.2+k3s1  
</code></pre>
</div>
</div>
<p id="bkmrk-then-test-the-health">Then test the health of the cluster pods:</p>
<div id="bkmrk-sudo-k3s-kubectl-get-0">
<div>
<pre><code>sudo k3s kubectl get pods --all-namespaces
</code></pre>
</div>
</div>
<p id="bkmrk-result%3A%C2%A0you-have-suc"><strong>Result:</strong> You have successfully set up a K3s Kubernetes cluster.</p>
<h3 id="bkmrk-3.-save-and-start-us">3. Save and Start Using the kubeconfig File</h3>
<p id="bkmrk-when-you-installed-k">When you installed K3s on each Rancher server node, a <code>kubeconfig</code> file was created on the node at <code>/etc/rancher/k3s/k3s.yaml</code>. This file contains credentials for full access to the cluster, and you should save this file in a secure location.</p>
<p id="bkmrk-to-use-this%C2%A0kubeconf">To use this <code>kubeconfig</code> file,</p>
<div id="bkmrk-install%C2%A0kubectl%2C%C2%A0a-k">
<div>
<ol>
<li>Install <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl">kubectl,</a> a Kubernetes command-line tool.</li>
<li>Copy the file at <code>/etc/rancher/k3s/k3s.yaml</code> and save it to the directory <code>~/.kube/config</code> on your local machine.</li>
<li>In the kubeconfig file, the <code>server</code> directive is defined as localhost. Configure the server as the DNS of your load balancer, referring to port 6443. (The Kubernetes API server will be reached at port 6443, while the Rancher server will be reached at ports 80 and 443.) Here is an example <code>k3s.yaml</code>:</li>
</ol>
<div>
<pre><code class="language-yml" data-lang="yml">apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: [CERTIFICATE-DATA]
    server: [LOAD-BALANCER-DNS]:6443 # Edit this line
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: [PASSWORD]
    username: admin</code></pre>
</div>
</div>
</div>
<p id="bkmrk-result%3A%C2%A0you-can-now-"><strong>Result:</strong> You can now use <code>kubectl</code> to manage your K3s cluster. If you have more than one kubeconfig file, you can specify which one you want to use by passing in the path to the file when using <code>kubectl</code>:</p>
<div id="bkmrk-kubectl---kubeconfig">
<div>
<pre><code>kubectl --kubeconfig ~/.kube/config/k3s.yaml get pods --all-namespaces
</code></pre>
</div>
</div>
<p id="bkmrk-for-more-information">For more information about the <code>kubeconfig</code> file, refer to the <a href="https://rancher.com/docs/k3s/latest/en/cluster-access/">K3s documentation</a> or the <a href="https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/">official Kubernetes documentation</a> about organizing cluster access using <code>kubeconfig</code> files.</p>
<h3 id="bkmrk-4.-check-the-health-">4. Check the Health of Your Cluster Pods</h3>
<p id="bkmrk-now-that-you-have-se">Now that you have set up the <code>kubeconfig</code> file, you can use <code>kubectl</code> to access the cluster from your local machine.</p>
<p id="bkmrk-check-that-all-the-r">Check that all the required pods and containers are healthy are ready to continue:</p>
<div id="bkmrk-ubuntu%40ip-172-31-60--0">
<div>
<pre><code>ubuntu@ip-172-31-60-194:~$ sudo kubectl get pods --all-namespaces
NAMESPACE       NAME                                      READY   STATUS    RESTARTS   AGE
kube-system     metrics-server-6d684c7b5-bw59k            1/1     Running   0          8d
kube-system     local-path-provisioner-58fb86bdfd-fmkvd   1/1     Running   0          8d
kube-system     coredns-d798c9dd-ljjnf                    1/1     Running   0          8d
</code></pre>
</div>
</div>
<p id="bkmrk-result%3A%C2%A0you-have-con"><strong>Result:</strong> You have confirmed that you can access the cluster with <code>kubectl</code> and the K3s cluster is running successfully. Now the Rancher management server can be installed on the cluster.</p>