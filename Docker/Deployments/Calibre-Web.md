---
title: Calibre-Web Deployment
description: 
published: true
date: 2022-03-23T19:18:09.818Z
tags: docker
editor: markdown
dateCreated: 2022-03-23T19:18:09.818Z
---

<p id="bkmrk-calibre-web-is-a-web"><a class="css-4rbku5 css-1dbjc4n r-1loqt21 r-1471scf r-1otgn73 r-1i6wzkk r-lrvibr" href="https://github.com/janeczku/calibre-web" data-rnw-int-class="link____"><span data-key="e37e0022d00d4e8884ebc60e0177627b" data-rnw-int-class="nearest_262-1645_264-1646-240__">Calibre-web</span></a><span data-key="d2a23b710b5a48d29843b61b5f0839fc" data-slate-fragment="JTdCJTIyb2JqZWN0JTIyJTNBJTIyZG9jdW1lbnQlMjIlMkMlMjJkYXRhJTIyJTNBJTdCJTdEJTJDJTIybm9kZXMlMjIlM0ElNUIlN0IlMjJvYmplY3QlMjIlM0ElMjJibG9jayUyMiUyQyUyMnR5cGUlMjIlM0ElMjJwYXJhZ3JhcGglMjIlMkMlMjJpc1ZvaWQlMjIlM0FmYWxzZSUyQyUyMmRhdGElMjIlM0ElN0IlN0QlMkMlMjJub2RlcyUyMiUzQSU1QiU3QiUyMm9iamVjdCUyMiUzQSUyMnRleHQlMjIlMkMlMjJsZWF2ZXMlMjIlM0ElNUIlN0IlMjJvYmplY3QlMjIlM0ElMjJsZWFmJTIyJTJDJTIydGV4dCUyMiUzQSUyMiUyMiUyQyUyMm1hcmtzJTIyJTNBJTVCJTVEJTJDJTIyc2VsZWN0aW9ucyUyMiUzQSU1QiU1RCU3RCU1RCUyQyUyMmtleSUyMiUzQSUyMmJhN2FkM2VkZGNkNjQxYTg5ZTZkMmZhZDE0YTVhZGMyJTIyJTdEJTJDJTdCJTIyb2JqZWN0JTIyJTNBJTIyaW5saW5lJTIyJTJDJTIydHlwZSUyMiUzQSUyMmxpbmslMjIlMkMlMjJpc1ZvaWQlMjIlM0FmYWxzZSUyQyUyMmRhdGElMjIlM0ElN0IlMjJyZWYlMjIlM0ElN0IlMjJraW5kJTIyJTNBJTIydXJsJTIyJTJDJTIydXJsJTIyJTNBJTIyaHR0cHMlM0ElMkYlMkZnaXRodWIuY29tJTJGamFuZWN6a3UlMkZjYWxpYnJlLXdlYiUyMiU3RCU3RCUyQyUyMm5vZGVzJTIyJTNBJTVCJTdCJTIyb2JqZWN0JTIyJTNBJTIydGV4dCUyMiUyQyUyMmxlYXZlcyUyMiUzQSU1QiU3QiUyMm9iamVjdCUyMiUzQSUyMmxlYWYlMjIlMkMlMjJ0ZXh0JTIyJTNBJTIyQ2FsaWJyZS13ZWIlMjIlMkMlMjJtYXJrcyUyMiUzQSU1QiU1RCUyQyUyMnNlbGVjdGlvbnMlMjIlM0ElNUIlNUQlN0QlNUQlMkMlMjJrZXklMjIlM0ElMjI2ZDlmZWJiMTNkODU0YWM0ODdkYWUyODA5MDIwNWM1ZSUyMiU3RCU1RCUyQyUyMmtleSUyMiUzQSUyMmUzN2UwMDIyZDAwZDRlODg4NGViYzYwZTAxNzc2MjdiJTIyJTdEJTJDJTdCJTIyb2JqZWN0JTIyJTNBJTIydGV4dCUyMiUyQyUyMmxlYXZlcyUyMiUzQSU1QiU3QiUyMm9iamVjdCUyMiUzQSUyMmxlYWYlMjIlMkMlMjJ0ZXh0JTIyJTNBJTIyJTIwaXMlMjBhJTIwd2ViJTIwYXBwJTIwcHJvdmlkaW5nJTIwYSUyMGNsZWFuJTIwaW50ZXJmYWNlJTIwZm9yJTIwYnJvd3NpbmclMkMlMjByZWFkaW5nJTIwYW5kJTIwZG93bmxvYWRpbmclMjBlQm9va3MlMjB1c2luZyUyMGFuJTIwZXhpc3RpbmclMjBDYWxpYnJlJTIwZGF0YWJhc2UuJTIwSXQlMjBpcyUyMGFsc28lMjBwb3NzaWJsZSUyMHRvJTIwaW50ZWdyYXRlJTIwZ29vZ2xlJTIwZHJpdmUlMjBhbmQlMjBlZGl0JTIwbWV0YWRhdGElMjBhbmQlMjB5b3VyJTIwY2FsaWJyZSUyMGxpYnJhcnklMjB0aHJvdWdoJTIwdGhlJTIwYXBwJTIwaXRzZWxmLiUyMiUyQyUyMm1hcmtzJTIyJTNBJTVCJTVEJTJDJTIyc2VsZWN0aW9ucyUyMiUzQSU1QiU1RCU3RCU1RCUyQyUyMmtleSUyMiUzQSUyMmQyYTIzYjcxMGI1YTQ4ZDI5ODQzYjYxYjVmMDgzOWZjJTIyJTdEJTVEJTJDJTIya2V5JTIyJTNBJTIyMjI5MWRhZDk5ZDI0NGFiYTkwZmZhY2NlZDBkZWZiZTElMjIlN0QlNUQlMkMlMjJrZXklMjIlM0ElMjIxNDg2YzEyNjQ2MzE0N2RmYjk5Yjk4ZDE5ZTkzNzY2NyUyMiU3RA=="> is a web app providing a clean interface for browsing, reading and downloading eBooks using an existing Calibre database. It is also possible to integrate google drive and edit metadata and your calibre library through the app itself.</span></p>
<h4 id="bkmrk-%E2%80%8Blinuxserver%2Fcalibre"><span data-key="892a2351b8ba4d329dfdbbdba0f14bc1">​</span><a class="css-4rbku5 css-1dbjc4n r-1loqt21 r-1471scf r-1otgn73 r-1i6wzkk r-lrvibr" href="https://github.com/linuxserver/docker-calibre-web" data-rnw-int-class="link____"><span data-key="234321ce9ae048aa93ccbfeeb19607e0" data-rnw-int-class="nearest_262-1645_264-1646-240__">linuxserver/calibre-web</span></a><span data-key="a124285be1fb4f71818447d8ebe0b703">​</span></h4>
<h3 id="bkmrk-usage" dir="auto">Usage</h3>
<h4 id="bkmrk-docker-compose-%28reco" dir="auto">docker-compose (recommended,&nbsp;<a href="https://docs.linuxserver.io/general/docker-compose" rel="nofollow">click here for more info</a>)</h4>
<div data-rnw-media-class="1495__1493-127">
<div data-rnw-media-class="1495__1493-127">
<div data-testid="page.contentEditor" data-slate-editor="true" data-key="c629250e671e442abefc8e72b2172daa" data-gramm="false">
<div>
<div data-key="dc5ba47f3db6443aba112dcb3ae73ae5">
<div data-block-content="dc5ba47f3db6443aba112dcb3ae73ae5">
<div>
<pre>---
<span class="pl-ent">version</span>: <span class="pl-s"><span class="pl-pds">"</span>2.1<span class="pl-pds">"</span></span>
<span class="pl-ent">services</span>:
  <span class="pl-ent">calibre-web</span>:
    <span class="pl-ent">image</span>: <span class="pl-s">lscr.io/linuxserver/calibre-web</span>
    <span class="pl-ent">container_name</span>: <span class="pl-s">calibre-web</span>
    <span class="pl-ent">environment</span>:
      - <span class="pl-s">PUID=1000</span>
      - <span class="pl-s">PGID=1000</span>
      - <span class="pl-s">TZ=Europe/London</span>
      - <span class="pl-s">DOCKER_MODS=linuxserver/calibre-web:calibre </span><span class="pl-c">#optional</span>
      - <span class="pl-s">OAUTHLIB_RELAX_TOKEN_SCOPE=1 </span><span class="pl-c">#optional</span>
    <span class="pl-ent">volumes</span>:
      - <span class="pl-s">/path/to/data:/config</span>
      - <span class="pl-s">/path/to/calibre/library:/books</span>
    <span class="pl-ent">ports</span>:
      - <span class="pl-c1">8083:8083</span>
    <span class="pl-ent">restart</span>: <span class="pl-s">unless-stopped</span></pre>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<h3 id="bkmrk-application-setup" dir="auto">Application Setup</h3>
<p id="bkmrk-webui-can-be-found-a" dir="auto">Webui can be found at&nbsp;<code>http://your-ip:8083</code></p>
<p id="bkmrk-on-the-initial-setup" dir="auto">On the initial setup screen, enter&nbsp;<code>/books</code>&nbsp;as your calibre library location.</p>
<p id="bkmrk-default-admin-login%3A" dir="auto"><strong>Default admin login:</strong>&nbsp;<em>Username:</em>&nbsp;admin&nbsp;<em>Password:</em>&nbsp;admin123</p>
<p id="bkmrk-unrar-is-included-by" dir="auto">Unrar is included by default and needs to be set in the Calibre-Web admin page (Basic Configuration:External Binaries) with a path of&nbsp;<code>/usr/bin/unrar</code></p>
<p id="bkmrk-x86-64-only%C2%A0we-have-" dir="auto"><strong>x86-64 only</strong>&nbsp;We have implemented the optional ability to pull in the dependencies to enable ebook conversion utilising Calibre, this means if you don't require this feature the container isn't uneccessarily bloated but should you require it, it is easily available. This optional layer will be rebuilt automatically on our CI pipeline upon new Calibre releases so you can stay up to date. To use this option add the optional environmental variable as detailed above to pull an addition docker layer to enable ebook conversion and then in the Calibre-Web admin page (Basic Configuration:External Binaries) set the&nbsp;<strong>Path to Calibre E-Book Converter</strong>&nbsp;to&nbsp;<code>/usr/bin/ebook-convert</code></p>
<p id="bkmrk-this-image-contains-" dir="auto">This image contains the&nbsp;<a href="https://pgaskin.net/kepubify/" rel="nofollow">kepubify</a>&nbsp;ebook conversion tool (MIT License) to convert epub to kepub. In the Calibre-Web admin page (Basic Configuration:External Binaries) set the&nbsp;<strong>Path to Kepubify E-Book Converter</strong>&nbsp;to&nbsp;<code>/usr/bin/kepubify</code></p>
<p id="bkmrk-to-reverse-proxy-wit" dir="auto">To reverse proxy with our Letsencrypt docker container we include a preconfigured reverse proxy config, for other instances of Nginx use the following location block:</p>
<div>
<pre><code>        location /calibre-web {
                proxy_pass              http://&lt;your-ip&gt;:8083;
                proxy_set_header        Host            $http_host;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header        X-Scheme        $scheme;
                proxy_set_header        X-Script-Name   /calibre-web;
        }
</code></pre>
</div>
<h3 id="bkmrk-%C2%A0" dir="auto">&nbsp;</h3>
<h3 id="bkmrk-parameters" dir="auto"><span style="color: #aaaaaa;">Error: Require Re-Authentication on Every Page Change</span></h3>
<p id="bkmrk-https%3A%2F%2Fgithub.com%2Fj"><span style="color: #aaaaaa;"><a href="https://github.com/janeczku/calibre-web/issues/1466">https://github.com/janeczku/calibre-web/issues/1466</a></span></p>
<p id="bkmrk-the-error-stems-from" dir="auto">The error stems from using Cloudflare DNS proxy to reach the web UI. The current image utilizes a security layer (Flask) that logs the session from a client. When proxied through Cloudlfare, the client session is altered on page change, requiring re-authentication.</p>
<p id="bkmrk-when-session-protect">When session protection is active, each request generates an identifier for the user&rsquo;s computer (basically, a secure hash of the IP address and user agent). If the session does not have an associated identifier, the one generated will be stored. If it has an identifier, and it matches the one generated, then the request is OK.</p>
<p id="bkmrk-if-the-identifiers-d">If the identifiers do not match in&nbsp;<code class="xref py py-obj docutils literal notranslate"><span class="pre">basic</span></code>&nbsp;mode, or when the session is permanent, then the session will simply be marked as non-fresh, and anything requiring a fresh login will force the user to re-authenticate. (Of course, you must be already using fresh logins where appropriate for this to have an effect.)</p>
<p id="bkmrk-if-the-identifiers-d-0">If the identifiers do not match in&nbsp;<code class="xref py py-obj docutils literal notranslate"><span class="pre">strong</span></code>&nbsp;mode for a non-permanent session, then the entire session (as well as the remember token if it exists) is deleted.</p>
<p id="bkmrk-for-cloudflare-users">For Cloudflare users, the ip address of the client is changing which triggers session invalidation.</p>
<h5 id="bkmrk-fix%3A">Fix:</h5>
<p id="bkmrk-downgrading-to-0.6.1" dir="auto">Downgrading to 0.6.12 fixes it. I'm using the docker image <code>ghcr.io/linuxserver/calibre-web:version-0.6.12</code>&nbsp;with the following sed replacements:</p>
<div data-rnw-media-class="1495__1493-127">
<div data-rnw-media-class="1495__1493-127">
<div data-testid="page.contentEditor" data-slate-editor="true" data-key="c629250e671e442abefc8e72b2172daa" data-gramm="false">
<div>
<div data-key="dc5ba47f3db6443aba112dcb3ae73ae5">
<div data-block-content="dc5ba47f3db6443aba112dcb3ae73ae5">
<div>
<pre><code>docker exec calibre-web find /app/calibre-web/cps/ -mindepth 1 -type f -exec sed -i 's/data-target="#bookDetailsModal"//g' {} \;
docker exec calibre-web find /app/calibre-web/cps/ -mindepth 1 -type f -exec sed -i "s/lm.session_protection = 'strong'/lm.session_protection = 'None'/g" {} \;</code></pre>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
<p id="bkmrk-staying-on-0.6.12-is" dir="auto">Staying on 0.6.12 is the second best solution, but it works for now.</p>
<h5 id="bkmrk-fix-2%3A%C2%A0" dir="auto">Fix 2:</h5>
<h5 dir="auto">(Haven't tried yet)</h5>
<p id="bkmrk-if-you-are-using-the" dir="auto">If you are using the linuxserver/calibre-web docker image then here is a workaround:</p>
<ol id="bkmrk-create-a-directory-n" dir="auto">
<li>Create a directory named&nbsp;<code>custom-cont-init.d</code>&nbsp;inside the&nbsp;<code>/config</code>&nbsp;mount point (so&nbsp;<code>/config/custom-cont-init.d</code>&nbsp;on the host)</li>
<li>Create a file named:&nbsp;<code>patch-session-protection.sh</code>&nbsp;in that directory</li>
<li>Add the contents</li>
</ol>
```
<div>
<pre><span class="pl-c">#!/bin/bash</span>

echo</span> <span class="pl-s"><span class="pl-pds">"</span>**** patching calibre-web - removing session protection ****<span class="pl-pds">"</span></span>

sed -i <span class="pl-s"><span class="pl-pds">"</span>/lm.session_protection = 'strong'/d<span class="pl-pds">"</span></span> /app/calibre-web/cps/__init__.py</pre>
</div>
```
<ol id="bkmrk-restart-your-contain" dir="auto" start="4">
<li>Restart your container</li>
</ol>
<div data-rnw-media-class="1495__1493-127">
<div data-testid="page.contentEditor" data-slate-editor="true" data-key="c629250e671e442abefc8e72b2172daa" data-gramm="false">
<div>
<div data-key="dc5ba47f3db6443aba112dcb3ae73ae5">
<div data-block-content="dc5ba47f3db6443aba112dcb3ae73ae5"></div>
</div>
</div>
<div>
<div data-key="d6a85a6cced644c58b379b4380b7a20e">
<div data-block-content="d6a85a6cced644c58b379b4380b7a20e">
<div></div>
</div>
</div>
</div>
</div>
</div>