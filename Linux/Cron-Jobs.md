---
title: Cron-Jobs
description: 
published: true
date: 2022-03-19T19:59:10.329Z
tags: linux
editor: markdown
dateCreated: 2022-03-19T19:59:08.341Z
---

<table id="bkmrk-cron-job-command-run">
<tbody>
<tr>
<td><strong>Cron Job</strong></td>
<td><strong>Command</strong></td>
</tr>
<tr>
<td><strong>Run Cron Job Every Minute</strong></td>
<td><strong><em>* * * * * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job Every 30 Minutes</strong></td>
<td><strong><em>30 * * * * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job Every Hour</strong></td>
<td><strong><em>0 * * * */root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job Every Day at Midnight</strong></td>
<td><strong><em>0 0 * * * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job at 2 am Every Day</strong></td>
<td><strong><em>0 2 * * * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job Every 1<sup>st</sup> of the Month</strong></td>
<td><strong><em>0 0 1 * * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job Every 15<sup>th</sup> of the Month</strong></td>
<td><strong><em>0 0 15 * * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job on December 1<sup>st</sup> –</strong><strong> Midnight</strong></td>
<td><strong><em>0 0 0 12 * /root/backup.sh</em></strong></td>
</tr>
<tr>
<td><strong>Run Cron Job on Saturday at Midnight</strong></td>
<td><strong><em>0 0 * * 6 /root/backup.sh</em></strong></td>
</tr>
</tbody>
</table>
<h3 id="bkmrk-strings-in-crontab">Strings in Crontab</h3>
<p id="bkmrk-strings-are-among-th">Strings are among the developer’s favorite things because they help to save time by eliminating repetitive writing. Cron has specific strings you can use to create commands quicker:</p>
<ol id="bkmrk-%40hourly%3A-run-once-ev">
<li><code>@hourly</code>: Run once every hour i.e. “<strong>0 * * * *</strong>“</li>
<li><code>@midnight</code>: Run once every day i.e. “<strong>0 0 * * *</strong>“</li>
<li><code>@daily</code>: same as midnight</li>
<li><code>@weekly</code>: Run once every week, i.e. “<strong>0 0 * * 0</strong>“</li>
<li><code>@monthly</code>: Run once every month i.e. “<strong>0 0 1 * *</strong>“</li>
<li><code>@annually</code>: Run once every year i.e. “<strong>0 0 1 1 *</strong>“</li>
<li><code>@yearly</code>: same as <strong>@annually</strong></li>
<li><code>@reboot</code>: Run once at every startup</li>
</ol>