---
title: "rclone container in Docker on Synology DSM7"
date: 2024-01-14T15:55:00-08:00
draft: false
tags: ["docker", "synology"]
---

I've been using Synology's in-built Cloud Sync for a while, but I noticed a few things about it today:
1. It seems to use an incredible amount of resources on the box
1. It won't let me add the same account twice so I can sync one particular thing to two places
1. It's missing Google Photos support (which I'm looking at for my [Photoprism](photoprism.md) organization project)

So I've decided to give [rclone](https://rclone.org) a try or at least get it installed.

### Setup Process:
1. Prerequisite: Have a folder somewhere on your Synology to keep all your Docker configs. Use `File Station` to set this up. Mine's at `/volume1/docker` and shows up as `docker`.
1. Open `Container Manager`.
1. Prerequisite: Have a private network established. This is likely done by default, but if not, go to the `Network` section with the left-side navigation tree. Mine's named `bridge` and has `IPv4 configuration` set to `Auto`, `IPv6 configuration` set to `None` and `Disable IP  Masquerade` as off.
1. Go to the `Registry` section with the left-side navigation tree.
1. Search for `rclone` in the top-right corner.
1. Double-click/download `rclone/rclone`. Choose `latest` as the tag if/when it asks.
1. Go to the `Container` section with the left-side navigation tree.
1. Click `Create`.
1. Choose from the `Image` dropdown and find `rclone/rclone:latest`.
1. Name the container `rclone` instead of the `rclone-rclone-1` it says by default as that's a mouthful.
1. Next.
1. Based on [this rclone forum post](https://forum.rclone.org/t/how-to-set-up-rclone-webgui-server-as-a-docker-container/14330), set the following advanced settings:
  - **Port Settings**: Hit `+ Add` then map `5572` on both local and remote side. Use `TCP`.
  - **Volume Settings**:
    - `+ Add Folder`: Create `docker/rclone/config` and map to `/config/rclone` with `Read/Write`
    - `+ Add Folder`: Create `docker/rclone/logs` and map to `/logs` with `Read/Write`
  - **Environment**:
    - `PATH` = `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`
    - `XDG_CONFIG_HOME` = `/config`
    - `PHP_TZ` = `America/Los_Angeles` (use [wikipedia](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to find matching zone).
    - `PUID` = `1026` (to find this and the below, check out [this blog post](https://blog.michelfailing.de/how-to-obtain-your-your-puid-and-pgid-on-a-synology-nas/))
    - `PGID` = `101`
  - **Network**: `bridge` from above or whatever it's named
  - **Execution Command**: `rcd --rc-web-gui --rc-addr :5572 --rc-user secret_user --rc-pass secret_password`
1. Finish and start it. Double-click it in the list and move to the `Log` tab to see what's going on inside.
1. Connect to `synology-ip:5572` to see the GUI and use `secret_user` and `secret_password` from above to log-in.

### Making Configurations:
It's very frustrating because it will be trying to launch a browser from the container for a bunch of stuff. I haven't figured out how to make it do the "headless" mode where it tells you to do a redirect.

Instead, I went to download the [Windows edition](https://rclone.org/downloads/) of `rclone` so I could set up the config locally then upload it to `docker/rclone/config` on the remote end and just use the GUI to monitor.

1. Download `rclone.exe` for Windows
1. Open a Terminal and switch to that directory after unpacking it.
1. Run `rclone config` to start adding configurations and logging in.
1. Move the configuration onto the Synology overwriting the one in `docker/rclone/config`.

### Finding the configuration:
Couldn't really find it easily by Googling so I installed [everything by voidtools](https://www.voidtools.com/) and searched for `rclone.conf`.

Turns out it's in `%APPDATA%\Roaming\rclone\rclone.conf`.

### Future:
- Presumably I will have to expose more directories to the container to enable it to sync things by adding more **Volume Settings**
- Mapping this in [Nginx Proxy Manager for Home Assistant](https://community.home-assistant.io/t/home-assistant-community-add-on-nginx-proxy-manager/111830) like I do other things... doesn't seem to want to work.
- The authentication uses both basic auth and then asks again for auth on a web page. Maybe I can turn off the first basic auth? That seems silly.
- This looks like it might be abandoned because the [Github Project repository](https://github.com/rclone/rclone-webui-react) hasn't been edited in 3-5 years... so I might just have to stick with Synology Cloud Sync or figure out how to script this CLI-style.
