---
title: Install/Upgrade Rancher on a Kubernetes Cluster
description: 
published: true
date: 2022-03-19T19:55:46.515Z
tags: kubernetes, rancher
editor: markdown
dateCreated: 2022-03-19T19:55:43.997Z
---

<h3 id="bkmrk-1.-add-the-helm-char">1. Add the Helm Chart Repository</h3>
<p id="bkmrk-use%C2%A0helm-repo-add%C2%A0co">Use <code>helm repo add</code> command to add the Helm chart repository that contains charts to install Rancher. For more information about the repository choices and which is best for your use case, see <a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/chart-options/#helm-chart-repositories">Choosing a Version of Rancher</a>.</p>
<pre id="bkmrk-helm-repo-add-ranche"><code class="language-">helm repo add rancher-stable https://releases.rancher.com/server-charts/stable</code></pre>
<h3 id="bkmrk-2.-create-a-namespac">2. Create a Namespace for Rancher</h3>
<p id="bkmrk-we%E2%80%99ll-need-to-define">We’ll need to define a Kubernetes namespace where the resources created by the Chart should be installed. This should always be <code>cattle-system</code>:</p>
<pre id="bkmrk-kubectl-create-names"><code class="language-">kubectl create namespace cattle-system</code></pre>
<h3 id="bkmrk-3.-choose-your-ssl-c">3. Choose your SSL Configuration</h3>
<p id="bkmrk-the-rancher-manageme">The Rancher management server is designed to be secure by default and requires SSL/TLS configuration.</p>
<blockquote id="bkmrk-note%3Aif-you-want-ter">
<p><strong>NOTE:</strong>If you want terminate SSL/TLS externally, see <a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/chart-options/#external-tls-termination">TLS termination on an External Load Balancer</a>.</p>
</blockquote>
<p id="bkmrk-there-are-three-reco">There are three recommended options for the source of the certificate used for TLS termination at the Rancher server:</p>
<ul id="bkmrk-rancher-generated-tl">
<li><strong>Rancher-generated TLS certificate:</strong> In this case, you will need to install <code>cert-manager</code> into the cluster. Rancher utilizes <code>cert-manager</code> to issue and maintain its certificates. Rancher will generate a CA certificate of its own, and sign a cert using that CA. <code>cert-manager</code> is then responsible for managing that certificate.</li>
<li><strong>Let’s Encrypt:</strong> The Let’s Encrypt option also uses <code>cert-manager</code>. However, in this case, cert-manager is combined with a special Issuer for Let’s Encrypt that performs all actions (including request and validation) necessary for getting a Let’s Encrypt issued cert. This configuration uses HTTP validation (<code>HTTP-01</code>), so the load balancer must have a public DNS record and be accessible from the internet.</li>
<li><strong>Bring your own certificate:</strong> This option allows you to bring your own public- or private-CA signed certificate. Rancher will use that certificate to secure websocket and HTTPS traffic. In this case, you must upload this certificate (and associated key) as PEM-encoded files with the name <code>tls.crt</code> and <code>tls.key</code>. If you are using a private CA, you must also upload that certificate. This is due to the fact that this private CA may not be trusted by your nodes. Rancher will take that CA certificate, and generate a checksum from it, which the various Rancher components will use to validate their connection to Rancher.</li>
</ul>
<table id="bkmrk-configuration-helm-c">
<thead>
<tr>
<th>CONFIGURATION</th>
<th>HELM CHART OPTION</th>
<th>REQUIRES CERT-MANAGER</th>
</tr>
</thead>
<tbody>
<tr>
<td>Rancher Generated Certificates (Default)</td>
<td><code>ingress.tls.source=rancher</code></td>
<td><a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/#4-install-cert-manager">yes</a></td>
</tr>
<tr>
<td>Let’s Encrypt</td>
<td><code>ingress.tls.source=letsEncrypt</code></td>
<td><a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/#4-install-cert-manager">yes</a></td>
</tr>
<tr>
<td>Certificates from Files</td>
<td><code>ingress.tls.source=secret</code></td>
<td>no</td>
</tr>
</tbody>
</table>
<h3 id="bkmrk-4.-install-cert-mana">4. Install cert-manager</h3>
<blockquote id="bkmrk-you-should-skip-this">
<p>You should skip this step if you are bringing your own certificate files (option <code>ingress.tls.source=secret</code>), or if you use <a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/chart-options/#external-tls-termination">TLS termination on an external load balancer</a>.</p>
</blockquote>
<p id="bkmrk-this-step-is-only-re">This step is only required to use certificates issued by Rancher’s generated CA (<code>ingress.tls.source=rancher</code>) or to request Let’s Encrypt issued certificates (<code>ingress.tls.source=letsEncrypt</code>).</p>
<div id="bkmrk-click-to-expand"><label for="cert-manager">CLICK TO EXPAND</label></div>
<h3 id="bkmrk-5.-install-rancher-w">5. Install Rancher with Helm and Your Chosen Certificate Option</h3>
<p id="bkmrk-the-exact-command-to">The exact command to install Rancher differs depending on the certificate configuration.</p>
<p id="bkmrk-this-option-uses%C2%A0cer">This option uses <code>cert-manager</code> to automatically request and renew <a href="https://letsencrypt.org/">Let’s Encrypt</a> certificates. This is a free service that provides you with a valid certificate as Let’s Encrypt is a trusted CA.</p>
<blockquote id="bkmrk-note%3Ayou-need-to-hav">
<p><strong>NOTE:</strong>You need to have port 80 open as the HTTP-01 challenge can only be done on port 80.</p>
</blockquote>
<p id="bkmrk-in-the-following-com">In the following command,</p>
<div id="bkmrk-hostname%C2%A0is-set-to-t">
<div>
<div>
<div>
<ul>
<li><code>hostname</code> is set to the public DNS record,</li>
<li>Set the <code>bootstrapPassword</code> to something unique for the <code>admin</code> user.</li>
<li><code>ingress.tls.source</code> is set to <code>letsEncrypt</code></li>
<li><code>letsEncrypt.email</code> is set to the email address used for communication about your certificate (for example, expiry notices)</li>
<li>Set <code>letsEncrypt.ingress.class</code> to whatever your ingress controller is, e.g., <code>traefik</code>, <code>nginx</code>, <code>haproxy</code>, etc.</li>
<li>If you are installing an alpha version, Helm requires adding the <code>--devel</code> option to the command.</li>
</ul>
<pre><code>helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set bootstrapPassword=admin \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
  --set letsEncrypt.ingress.class=nginx
</code></pre>
</div>
</div>
</div>
</div>
<p id="bkmrk-wait-for-rancher-to-">Wait for Rancher to be rolled out:</p>
<div id="bkmrk-kubectl--n-cattle-sy">
<div>
<div>
<pre><code>kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
</code></pre>
</div>
</div>
</div>
<p id="bkmrk-the-rancher-chart-co">The Rancher chart configuration has many options for customizing the installation to suit your specific environment. Here are some common advanced scenarios.</p>
<ul id="bkmrk-http-proxy-private-c">
<li><a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/chart-options/#http-proxy">HTTP Proxy</a></li>
<li><a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/chart-options/#private-registry-and-air-gap-installs">Private container image Registry</a></li>
<li><a href="https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/chart-options/#external-tls-termination">TLS Termination on an External Load Balancer</a></li>
</ul>
<p id="bkmrk-see-the%C2%A0chart-option">See the <a href="https://rancher.com/docs/rancher/v2.6/en/installation/resources/chart-options/">Chart Options</a> for the full list of options.</p>
<h3 id="bkmrk-6.-verify-that-the-r">6. Verify that the Rancher Server is Successfully Deployed</h3>
<p id="bkmrk-after-adding-the-sec">After adding the secrets, check if Rancher was rolled out successfully:</p>
<pre id="bkmrk-kubectl--n-cattle-sy-0"><code class="language-">kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out</code></pre>
<p id="bkmrk-if-you-see-the-follo">If you see the following error: <code>error: deployment "rancher" exceeded its progress deadline</code>, you can check the status of the deployment by running the following command:</p>
<pre id="bkmrk-kubectl--n-cattle-sy-1"><code class="language-">kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m</code></pre>
<p id="bkmrk-it-should-show-the-s">It should show the same count for <code>DESIRED</code> and <code>AVAILABLE</code>.</p>
<h3 id="bkmrk-7.-save-your-options">7. Save Your Options</h3>
<p id="bkmrk-make-sure-you-save-t">Make sure you save the <code>--set</code> options you used. You will need to use the same options when you upgrade Rancher to new versions with Helm.</p>
<h3 id="bkmrk-finishing-up">Finishing Up</h3>
<p id="bkmrk-that%E2%80%99s-it.-you-shoul">That’s it. You should have a functional Rancher server.</p>
<p id="bkmrk-in-a-web-browser%2C-go">In a web browser, go to the DNS name that forwards traffic to your load balancer. Then you should be greeted by the colorful login page.</p>
<p id="bkmrk-doesn%E2%80%99t-work%3F-take-a">Doesn’t work? Take a look at the <a href="https://rancher.com/docs/rancher/v2.6/en/installation/resources/troubleshooting/">Troubleshooting</a> Page</p>