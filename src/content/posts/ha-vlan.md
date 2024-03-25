---
title: "Home Assistant crosses VLAN for Matter over Thread"
date: 2024-03-24T20:08:00-07:00
draft: false
tags: ["home assistant", "thread", "opnsense", "matter", "nanoleaf", "aqara"]
---

For Christmas, someone got me an [Aqara Door and Window Sensor P2](https://www.aqara.com/en/product/door-and-window-sensor-p2). It's a Matter over Thread device. The first one I have. The only other Matter devices I have so far are Matter over Wi-Fi ([Nanoleaf Essentials Holiday String Lights](https://nanoleaf.me/en-US/products/seasonal/holiday-string-lights/)) and two Wi-Fi-connected-but-Thread-gateways in the form of [Nanoleaf Shapes](https://nanoleaf.me/en-US/products/nanoleaf-shapes/?category=hexagons&pack=smarter-kit&size=7).

When I attempted to pair it with my Home Assistant... it couldn't correctly pair with the device.

After like 7 tries... I went to the Internet. And figured out that it's likely because I have the Nanoleaf Shapes Thread gateways sitting on my IOT stuff VLAN 80 and not on the main network where the Home Assistant Blue is sitting.

Note that I do allow all the IOT devices through the OPNsense firewall to talk to the Home Assistant Blue on the main LAN at 192.168.1.5... AND I have the [OPNsense Multicast DNS Proxy](https://docs.opnsense.org/manual/how-tos/multicast-dns.html) installed and configured to both **LAN** and **IOT**. So it was able to do the discovery process and see mDNS responses across the segments. But then I stumbled across [this forum post](https://community.home-assistant.io/t/thread-matter-router-rules-and-firewalls/583774) that says the important part... **Thread is link-local**. It uses link-local only IPv6 addresses... so it won't cross the VLAN boundary. The discovery can cross... but the remaining communication cannot. So that's why I couldn't finish the connection.

I wanted my Home Assistant instance to sit on both the main network and the VLAN so I could get Thread working with minimal additional reconfiguration.

My network looks like this:
- Main: 192.168.0.0/22 with the Home Assistant Blue sitting at 192.168.1.5
- IOT: 192.168.80.0/22 with VLAN tag 80 and IOT devices sprayed from 192.168.80.100 all the way up to the end of the range.

To solve this, I put the **Advanced SSH & Web Terminal** add-on into Home Assistant. I set it to *Show in sidebar* mode, I added a password on the config page (below `username: hassio` is a `password:` field that must be filled in) so I could start it, I turned on the flag to show it in the sidebar, then I started the add-on and went to the sidebar.

At the command promt, I typed `ha network info` to see what the existing network looked like:
```
docker:
    address: 172.30.32.0/23
    dns: 172.30.32.3
    gateway: 172.30.32.1
    interface: hassio
host_internet: true
interfaces:
- connected: true
  enabled: true
  interface: end0
  ipv4:
    address:
    - 192.168.1.5/22
    gateway: 192.168.0.1
    method: static
    nameservers:
    - 192.168.0.1
    ready: true
  ipv6:
    <redacted>
  mac: <redacted>
  primary: true
  type: ethernet
  vlan: null
  wifi: null
supervisor_internet: true
```

Then I ran this command to add the vlan
`ha network vlan end0 80 --ipv4-method static --ipv4-address 192.168.80.5/22 --ipv4-nameserver 192.168.80.1 --ipv4-gateway 192.168.80.1`

- 80 is the vlan tag number
- end0 is the interface name from the information command that I wanted to add the vlan on
- 192.168.80.5 is the static IP I wanted set 
- 22 is the size of this network. It's a 255.255.252.0 netmask and runs from 192.168.80.0 to 192.168.83.255
- 192.168.80.1 is the address on my router/gateway that also contains DNS services

When done, I saw this:
```
docker:
    address: 172.30.32.0/23
    dns: 172.30.32.3
    gateway: 172.30.32.1
    interface: hassio
host_internet: true
interfaces:
- connected: true   
  enabled: true
  interface: end0.80
  ipv4:
    address:
    - 192.168.80.5/22
    gateway: 192.168.80.1
    method: static
    nameservers: []
    ready: true
  ipv6:
    <redacted, but only private link-local addresses>
  mac: <redacted>
  primary: false
  type: vlan
  vlan:
    id: 80
    parent: <redacted guid>
  wifi: null
- connected: true
  enabled: true
  interface: end0
  ipv4:
    address:
    - 192.168.1.5/22
    gateway: 192.168.0.1
    method: static
    nameservers:
    - 192.168.0.1
    ready: true
  ipv6:
    <redacted>
  mac: <redacted>
  primary: true
  type: ethernet
  vlan: null
  wifi: null
supervisor_internet: true
```

Then I turned off, disabled, and uninstalled the **Advanced SSH & Web Terminal** add-on.

Now I went back to the GUI and it shows up. Go under *Settings > System > Network* and scroll down to *Configure network interfaces* (Ignore the `wlan0`... I thought I was going to do that but if this works via wired... that's even better.)

![A screenshot of the 'Configure network interfaces' page from the Home Assisstant UI per the description above. The headings show 'end0', 'wlan0', and 'end0.80'. The 'end0.80' heading is selected. A slide out menu for 'IPv4' is expanded while the lower 'IPv6' is collapsed. Under 'IPv4', the center radio button is set to 'Static' while the left 'Automatic' and the right 'Disabled' are not used. Below is a field labeled 'IP address/Netmask' with the value '192.168.80.5/22'. Below that is a field labeled 'Gateway address" with the value '192.168.80.1'. Below that is a field labeled 'DNS Servers' with the value '192.168.80.1'. Finally a 'Save' button in the lower right is grayed as disabled and there are an expansion of three dots in the lower left corner.](/2024-03-24-ha-vlan-80.png)

This is inspired by [this forum post entitled 'Setup VLAN and HA tutorial'](https://community.home-assistant.io/t/setup-vlan-and-ha-tutorial/87705/171) on the official Home Assistant community site.

Once this was all done, I did a full reboot of the entire Home Assistant Blue just figuring that attempting to do it live was pressing my luck.

Finally, I went back to the **Aqara** sensor and I tried pairing again. I went to my phone (Pixel 7 Pro) and opened the **Home Assistant* app. In the hamburger menu, I went to *Settings > Devices & Services* then clicked the *ADD INTEGRATION* button in the lower right. I chose *Add Matter device* and the Android shell took over. I set the Aqara device into listen mode by popping the battery in and holding the button until the blue light blinked. Then I scanned the Matter QR code on the back of the manual and waited. After about a minute or two... success.

And then I went to check the log in the *Matter Server* add-on for Home Assistant and saw this beautiful sight:

```
2024-03-24 20:37:59 (MainThread) INFO [matter_server.server.device_controller] Starting Matter commissioning using Node ID 8 and IP fd6d:8abb:d:1:2372:2d7b:fe:58c9 (attempt 1/3).
2024-03-24 20:38:06 (MainThread) INFO [matter_server.server.device_controller] Matter commissioning of Node ID 8 successful.
2024-03-24 20:38:06 (MainThread) INFO [matter_server.server.device_controller] Interviewing node: 8
2024-03-24 20:38:11 (MainThread) INFO [matter_server.server.device_controller.node_8] Setting-up node...
2024-03-24 20:38:11 (MainThread) INFO [matter_server.server.device_controller.node_8] Setting up attributes and events subscription.
2024-03-24 20:38:19 (MainThread) INFO [matter_server.server.device_controller.node_8] Subscription succeeded
2024-03-24 20:38:19 (MainThread) INFO [matter_server.server.device_controller] Commissioning of Node ID 8 completed.
```

It works!
