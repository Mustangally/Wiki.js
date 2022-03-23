---
title: Watchtower Deployment
description: 
published: true
date: 2022-03-23T19:12:45.070Z
tags: docker
editor: markdown
dateCreated: 2022-03-23T19:12:45.070Z
---

<p id="bkmrk-a-question-that-has-">A question that has always bothered me was how I should deal with security updates on applications that are running in docker containers. Security updates are often done automatically in Linux distros and that for a good reason. This is easily done via the package managers and there are many services and apps to automate package updates for you. But as docker containers are isolated from the core system and they are based on images, you need to come up with a better solution for these. With Watchtower, you can schedule container updates and minimize the downtime during the update.</p>
<h2 id="bkmrk-update-docker-contai">Update docker container automatically with Watchtower</h2>
<p id="bkmrk-watchtower-is-an-app">Watchtower is an application that will monitor all running docker containers. Once it discovers a change in the image it will pull down the new version automatically and restart the container with the new image. It is an open-source project you can find on GitHub and I’m using this in some installations. It offers you some configuration to schedule the update sequence and includes or excludes specific containers on the server. You can also add a notification that will be able to send you an email. Watchtower itself is also running in a docker container and configured by changing environment variables or command arguments.</p>
<h2 id="bkmrk-deploy-watchtower-vi">Deploy Watchtower via Docker-Compose</h2>
<p id="bkmrk-with-watchtower-you-">With watchtower you can update the running version of your containerized app simply by pushing a new image to the Docker Hub or your own image registry. Watchtower will pull down your new image, gracefully shut down your existing container and restart it with the same options that were used when it was deployed initially. Run the watchtower container with the following command:</p>
<pre id="bkmrk-version%3A-%223%22-service"><code class="language-">version: "3"
services:
watchtower:
image: containrrr/watchtower volumes:
- /var/run/docker.sock:/var/run/docker.sock</code></pre>
<h2 id="bkmrk-deploy-watchtower-vi-0">Deploy Watchtower via Docker CLI</h2>
<p id="bkmrk-watchtower-can-be-ea">Watchtower can be easily deployed by executing a simple docker run command.</p>
<div id="bkmrk-docker-run---name-wa">
<div>
<pre class="wp-block-code  language-bash" tabindex="0"><code class="  language-bash" lang="bash">docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower</code></pre>
</div>
</div>
<p id="bkmrk-you-might-wonder-why">You might wonder why there is no log output apart from the welcome message. If you want to increase the logging level or watchtower, you simply just add an argument.</p>
<div id="bkmrk-docker-run---name-wa-0">
<div>
<pre class="wp-block-code  language-bash" tabindex="0"><code class="  language-bash" lang="bash">docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --debug</code></pre>
</div>
</div>
<p id="bkmrk-this-will-automatica">This will automatically run once every day, but you can also do a quick test run with the following command. The “run-once” argument will only run the container once and then immediately exits.</p>
<div id="bkmrk-docker-run---name-wa-1">
<div>
<pre class="wp-block-code  language-bash" tabindex="0"><code class="  language-bash" lang="bash">docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --run-once --debug</code></pre>
</div>
</div>
<p id="bkmrk-note%2C-if-you-specify">Note, if you specify a version tag in the docker images, watchtower will not update the container to a newer version. For example, if you run nginx:1.18, watchtower will not upgrade it to nginx:1.19. If you want to stick a specific version of an application you can simply just specify the version in the tag. For all other containers, I would recommend using the “latest” tag, which is the default by the way.</p>
<h2 id="bkmrk-include-or-exclude-d">Include or Exclude docker containers from the update</h2>
<p id="bkmrk-but-sometimes%2C-you-w">But sometimes, you want to have the latest version of an image, but you still want to do updates manually for some containers. Or you may want to schedule the update to a specific date or time because you want to avoid downtime. You can include or exclude specific containers from the update procedure with labels. This means you don’t attach the label on the watchtower container, but you can attach the exclude or monitor-only label to other containers.</p>
<p id="bkmrk-let%E2%80%99s-run-an-nginx-s">Let’s run an Nginx server and exclude it from our update procedure.</p>
<div id="bkmrk-docker-run--d---labe">
<div>
<pre class="wp-block-code  language-bash" tabindex="0"><code class="  language-bash" lang="bash">docker run -d --label<span class="token operator">=</span>com.centurylinklabs.watchtower.enable<span class="token operator">=</span> <span class="token boolean">false</span> nginx</code></pre>
</div>
</div>
<p id="bkmrk-if-we-now-run-our-wa">If we now run our watchtower container, you can see it doesn’t reveal any running containers, because we have excluded the Nginx container.</p>
<div id="bkmrk-">
<div>

</div>
</div>
<h2 id="bkmrk-scheduled-updates-an">Scheduled Updates and clean up old images</h2>
<p id="bkmrk-you-can-also-schedul">You can also schedule the update procedure for a specific time. This uses the argument “–schedule” with a crontab argument. For example, if you want to run the update procedure every day at 4 am, use the following command.</p>
<div id="bkmrk-docker-run---name-wa-2">
<div>
<pre class="wp-block-code  language-bash" tabindex="0"><code class="  language-bash" lang="bash">docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped containrrr/watchtower --schedule <span class="token string">"0 0 4 * * *"</span> --debug</code></pre>
</div>
</div>
<p id="bkmrk-i-would-also-recomme">I would also recommend adding two more arguments to the command. The first one “—cleanup” will delete all old docker images. Once Watchtower has updated a container the old image will be still on the server. With the cleanup command, you can get rid of these orphaned images. The second one “–rolling-restart” will update one image at a time instead of stopping and starting all at once. This is very useful if you want to minimize the downtime.</p>
<p id="bkmrk-of-course%2C-there-are">Of course, there are also other arguments and improvements, you can find in <a href="https://containrrr.dev/watchtower/arguments/">the official documentation</a>.</p>