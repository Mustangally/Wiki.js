---
title: Connect Joplin to Nextcloud
description: 
published: true
date: 2022-05-05T15:02:23.770Z
tags: 
editor: markdown
dateCreated: 2022-05-05T15:02:23.770Z
---

# Connecting Joplin to Nextcloud
Step by Step Guide to get Joplin synced to Nextcloud.

## Install and Configure Joplin Sync Settings

Once you have Joplin installed (download from website and install), you just need to go to `tools`>`options`>`Synchronisation`

In this menu, select "Nextcloud" as the Syncronisation Target, then input the WebDAV URL. This will be: `https://YOURNEXTCLOUDURL/remote.php/webdav/Joplin`
Note that the `/Joplin` at the end is in reference to a file folder you've created in Nextcloud.

## Sign In

Next we'll need to authenticate to Nextcloud. To do this, we're going to use an API key password generated in Nextcloud. To do this, click on your user, go to `settings`, `security`, then scroll down to the bottom where you'll find `Devices & Sessions`. Enter in a session name in the text box, and click `generate new app password`. This will give you a username and password to enter into Joplin.

Once you've copied the credentials over, click on `Check synchronisation configuration`, and it should say Success.