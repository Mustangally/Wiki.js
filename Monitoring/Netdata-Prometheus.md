---
title: Monitor Your System with Grafana using Netdata and Prometheus
description: 
published: true
date: 2022-03-19T20:11:33.540Z
tags: monitoring
editor: markdown
dateCreated: 2022-03-19T20:11:31.526Z
---

<p id="bkmrk-the-idea-behind-this">The idea behind this project is to have a custom dashboard that displays only things you really want to see at a glance. You can choose to dig through the Prometheus Netdata metrics more to display anything you wish but below is the example I setup for this tutorial.</p>
<p id="bkmrk-prometheus-does-all-">Prometheus does all the work. The only thing required to be installed on the machine being monitored is Netdata. No other agents or workers are needed that could utilize more system memory on your servers. That's the beauty of Prometheus. Rather than coming to your machine to collect data, Prometheus waits for data to come to it instead, using API calls.</p>
<p id="bkmrk-"><img src="https://snip.lol/JipE3/WobiLulu65/raw.png"></p>
<p id="bkmrk-netdata-is-required-" class="callout info">Netdata is required to be installed on any machine that will be monitored. Please see <a href="https://www.youtube.com/watch?v=-ILREKzj_YA&amp;ab_channel=Geeked" target="_blank" rel="noopener">this video</a> before moving forward here.</p>
<p id="bkmrk-i-chose-to-host-graf">I chose to host Grafana and Prometheus on their own separate LXCcontainer using Proxmox. You can use a RPi or any host you wish. I do this so I can use one central host to keep things more organized. You only need one host for Grafana and Prometheus.</p>
<h3 id="bkmrk-install-grafana">Install Grafana</h3>
<p id="bkmrk-i-use-a-docker-stack">I use a docker stack through Portainer.</p>
<pre id="bkmrk-version%3A-%273.3%27-servi"><code class="language-">version: '3.3'
services:
    grafana:
        ports:
            - '3000:3000'
        container_name: grafana
        image: grafana/grafana</code></pre>
<p id="bkmrk-grafana-will-then-be">Grafana will then be accessible on port 3000. Login with admin/admin then choose a stronger password.</p>
<p id="bkmrk--0"><img src="https://snip.lol/JipE3/ZAmeJIXe49/raw.png"></p>
<h3 id="bkmrk-install-prometheus">Install Prometheus</h3>
<p id="bkmrk-again%2C-i-use-a-docke">Again, I use a docker stack through Portainer.</p>
<pre id="bkmrk-version%3A-%273.3%27-servi-0"><code class="language-">version: '3.3'
services:
    prometheus:
        ports:
            - '9090:9090'
        volumes:
            - '/docker/prometheus:/etc/prometheus'
        image: prom/prometheus</code></pre>
<p id="bkmrk-this-will-create-a-f">This will create a folder on your host machine at /docker/prometheus</p>
<p id="bkmrk-prometheus-will-then">Prometheus will then be accessible on port 9090.</p>
<p id="bkmrk--1"><a href="https://snip.lol/JipE3/GilUsaho86/raw.png" target="_blank" rel="noopener"><img src="https://snip.lol/JipE3/GilUsaho86/raw.png"></a></p>
<h3 id="bkmrk-create-the-prometheu">Create the prometheus.yml file</h3>
<pre id="bkmrk-cd-%2Fdocker%2Fprometheu"><code class="language-">cd /docker/prometheus</code></pre>
<pre id="bkmrk-touch-prometheus.yml"><code class="language-">touch prometheus.yml</code></pre>
<h3 id="bkmrk-edit-the-prometheus.">Edit the prometheus.yml file</h3>
<pre id="bkmrk-nano-%2Fdocker%2Fprometh"><code class="language-">nano /docker/prometheus/prometheus.yml</code></pre>
<p id="bkmrk-more-coming-soon%21">Below is my example prometheus.yml file. You should change the IP to match that of your Netdata web UI. You should not have to change anything above the pound line, only that in between.</p>
<pre id="bkmrk-%23-my-global-config-g"><code class="language-"># my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=&lt;job_name&gt;` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
    
#################################################################################################

  - job_name: 'netdata-scrape'

    metrics_path: '/api/v1/allmetrics'
    params:
      # format: prometheus | prometheus_all_hosts
      # You can use `prometheus_all_hosts` if you want Prometheus to set the `instance` to your hostname instead of IP 
      format: [prometheus]
      #
      # source: as-collected | raw | average | sum | volume
      # default is: average
      #source: [as-collected]
      #
      # server name for this prometheus - the default is the client IP
      # for Netdata to uniquely identify it
      #server: ['prometheus1']
    honor_labels: true
    static_configs:
      - targets: ['192.168.1.13:19999']
      
################################################################################################# </code></pre>
<p id="bkmrk-if-you-decide-to-mon">If you decide to monitor more than one Netdata instance just copy and paste from each pound line then edit the job name and target IP.</p>
<h3 id="bkmrk-%C2%A0">Prometheus Netdata Metrics</h3>
<p id="bkmrk-you-can-browse-the-m">You can browse the metrics explorer by pressing the small globe icon next to the search bar or use the metrics I used in the video to start your dashboard. You can see those below.</p>
<p id="bkmrk-on-the-prometheus-we">On the Prometheus webui, copy and paste the following metrics in the finder then copy the query you wish to display in Grafana.</p>
<p id="bkmrk--2"><a href="https://snip.lol/JipE3/yUliDOsI32/raw.png" target="_blank" rel="noopener"><img src="https://snip.lol/JipE3/yUliDOsI32/raw.png"></a></p>
<p id="bkmrk-system-uptime">System Uptime</p>
<pre id="bkmrk-netdata_system_uptim"><code class="language-">netdata_system_uptime_seconds_average</code></pre>
<p id="bkmrk-when-adding-this-to-">When adding this to Grafana be sure to select "seconds (s)" as the unit of measurement under Standard options.</p>
<p id="bkmrk-cpu-temperature-%28req">CPU Temperature (requires sensors to be installed on the server using lm-sensors)</p>
<pre id="bkmrk-netdata_sensors_temp"><code class="language-">netdata_sensors_temperature_Celsius_average</code></pre>
<p id="bkmrk-when-adding-this-to--0">When adding this to Grafana be sure to select "Celsius (°C)" as the unit of measurement under Standard options.</p>
<p id="bkmrk-cpu-usage">CPU Usage</p>
<pre id="bkmrk-netdata_cpu_cpu_perc"><code class="language-">netdata_cpu_cpu_percentage_average</code></pre>
<p id="bkmrk-when-adding-this-to--1">When adding this to Grafana be sure to select "Percent (0-100)" as the unit of measurement under Standard options.</p>
<p id="bkmrk-memory-used">Memory Used</p>
<p id="bkmrk-this-was-a-tricky-on">This was a tricky one. I had to take total memory and subtract the available memory from it to get an accurate number.</p>
<pre id="bkmrk-31970---netdata_mem_"><code class="language-">31970 - netdata_mem_available_MiB_average{instance="192.168.1.13:19999", job="netdata-scrape"}</code></pre>
<p id="bkmrk-when-adding-this-to--2">When adding this to Grafana be sure to select "mebibytes" as the unit of measurement under Standard options.</p>
<p id="bkmrk-as-you-can-see%2C-i-ha">As you can see, I have 32GB of RAM. I had to play with the numbers and used <a href="https://thehomelab.wiki/books/monitoring/page/install-bpytop-for-monitoring-linux" target="_blank" rel="noopener">bpytop</a> to compare. I was able to get the RAM spot on to match the same output in bpytop.</p>
<p id="bkmrk--3"><img src="https://snip.lol/JipE3/Jegelara42/raw.png"></p>
<p id="bkmrk--4"><img src="https://snip.lol/JipE3/ConAHaSI39/raw.png"></p>
<p id="bkmrk-hard-drive-space">Hard Drive Space</p>
<pre id="bkmrk-netdata_disk_space_g"><code class="language-">netdata_disk_space_GiB_average</code></pre>
<p id="bkmrk-when-adding-this-to--3">When adding this to Grafana be sure to select "gibibytes" as the unit of measurement under Standard options.</p>