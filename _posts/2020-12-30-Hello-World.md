---
layout: post
title: Raspberry Pi Setup
---

A step-by-step guide for setting up Deluge, Plex, etc. on the Raspberry Pi 4.

![https://deluge-torrent.org/](https://deluge-torrent.org/images/droplet.png)

## Headless
Setup a headless Raspian Lite installation.

### OS
Use the Raspberri Pi Imager to format and install Raspian Lite on a SD card.

### SSH
[Prepare SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md) (Only step 3)

Add empty `ssh` file to SD card. 

On windows: Right click in SD card folder -> New -> Text File -> Rename to `ssh`

### (Wifi)
[Prepare wifi](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)

Add `wpa_supplicant.conf` file to SD card. For Denmark use the country code `DK`:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=DK

network={
ssid="<Name of your wireless LAN>"
psk="<Password for your wireless LAN>"
}
```