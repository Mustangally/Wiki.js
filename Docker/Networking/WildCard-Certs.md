---
title: Wild Card Certs Using Traefik
description: 
published: true
date: 2022-03-23T19:20:44.091Z
tags: docker, networking, treafik
editor: markdown
dateCreated: 2022-03-23T19:20:44.091Z
---

<h2 id="bkmrk-traefik">Traefik</h2>
<pre id="bkmrk-mkdir-traefik-cd-tra"><code class="language-plaintext">mkdir traefik cd traefik mkdir data cd data touch acme.json chmod 600 acme.json touch traefik.yml </code></pre>
<p id="bkmrk-traefik.config-can-b"><code>traefik.config</code> can be found <a href="https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/traefik">here</a></p>
<p id="bkmrk-create-docker-networ">create docker network</p>
<pre id="bkmrk-docker-network-creat"><code class="language-plaintext">docker network create proxy </code></pre>
<pre id="bkmrk-touch-docker-compose"><code class="language-plaintext">touch docker-compose.yml </code></pre>
<p id="bkmrk-docker-compose.yml-c"><code>docker-compose.yml</code> can be found <a href="https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/traefik">here</a></p>
<pre id="bkmrk-cd-data-touch-config"><code class="language-plaintext">cd data touch config.yml </code></pre>
<pre id="bkmrk-docker-compose-up--d"><code class="language-plaintext">docker-compose up -d </code></pre>
<h2 id="bkmrk-portainer">Portainer</h2>
<pre id="bkmrk-mkdir-portainer-cd-p"><code class="language-plaintext">mkdir portainer cd portainer touch docker-compose.yml mkdir data </code></pre>
<p id="bkmrk-docker-compose.yml-c-0"><code>docker-compose.yml</code> can be found <a href="https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/portainer">here</a></p>
<h3 id="bkmrk-generate-basic-auth-">Generate Basic Auth Password</h3>
<pre id="bkmrk-sudo-apt-update-sudo"><code class="language-plaintext">sudo apt update sudo apt install apache2-utils </code></pre>
<pre id="bkmrk-echo-%24%28htpasswd--nb-"><code class="language-plaintext">echo $(htpasswd -nb &lt;USER&gt; &lt;PASSWORD&gt;) | sed -e s/\\$/\\$\\$/g </code></pre>
<p id="bkmrk-note%3A-replace-%3Cuser%3E">NOTE: Replace <code>&lt;USER&gt;</code> with your username and <code>&lt;PASSWORD&gt;</code> with your password to be hashed.</p>
<p id="bkmrk-paste-the-output-in-">Paste the output in your <code>docker-compose.yml</code> in line (<code>traefik.http.middlewares.traefik-auth.basicauth.users=&lt;USER&gt;:&lt;HASHED-PASSWORD&gt;</code>)</p>
<h4 id="bkmrk-spin-up-the-containe">Spin up the container</h4>
<pre id="bkmrk-docker-compose-up--d-0"><code class="language-plaintext">docker-compose up -d </code></pre>
<h2 id="bkmrk-traefik-routes-confi">Traefik Routes Config</h2>
<pre id="bkmrk-cd-traefik%2Fdata-nano"><code class="language-plaintext">cd traefik/data nano config.yml </code></pre>
<p id="bkmrk-config.yml-here"><code>config.yml</code> <a href="https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/traefik">here</a></p>
<pre id="bkmrk-docker-compose-up--d-1"><code class="language-plaintext">docker-compose up -d --force-recreate </code></pre>
<h2 id="bkmrk-common-error%3A-bad-ga">Common error: ERR_TOO_MANY_REDIRECTS</h2>
<p id="bkmrk-this-error-happens-w">This error happens when trying to proxy the DNS entry in Cloudflare DNS settings.</p>
<p id="bkmrk-to-resolve%2C-simply-c">To resolve, simply change Cloudflare's TLS/SSL Settings to "Full"</p>