---
layout: post
title: Raspberry Pi Setup
---

A step-by-step guide for setting up Deluge, Plex, etc. on the Raspberry Pi 4.

![https://deluge-torrent.org/]({{ site.baseurl }}/images/moviestarr.jpg)

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
## Installation
```
sudo apt update && sudo apt upgrade -y
```
### Deluge
[Instructions](https://sbcguides.com/install-deluge-on-raspberry-pi/)

Instead of using the group _debian-deluged_ for the services, I created a new group with `sudo groupadd media` and used this in both systemd services. Remember to add the _pi_ user to this group:
```
sudo usermod -a -G media pi
```
(You need to reboot for this to take effect)

Then do the following:
- Go to deluge web interface
- Change password
- Disable Network -> random incoming ports
- Disable Network -> UPnP, NAT-PMP and LSD
- Enable Daemon -> Allow remote connections
- (Change download location. Remember to `chown` and `chmod` the folder with the media group AND restart the deluged service afterwards!)

### VPN Client (PIA)
[Instructions](https://dotslashnotes.wordpress.com/2013/08/05/how-to-set-up-a-vpn-private-internet-access-in-raspberry-pi/)

Test if it works:
- `sudo openvpn --daemon --config denmark.ovpn`
- `ping google.com`

If "ping: google.com: Temporary failure in name resolution", then change DNS:

```
sudo nano /etc/dhcpcd.conf
```
Change this line:
```
static domain_name_servers=8.8.4.4 8.8.8.8
```
Save the file and restart:
```
sudo service dhcpcd restart
```

Connect OpenVPN on startup: https://www.ivpn.net/knowledgebase/linux/linux-autostart-openvpn-in-systemd-ubuntu/

Check torrent ip:
https://torguard.net/checkmytorrentipaddress.php

### NFS
For deluge to download to a (Synology) NFS network drive do the following:
- Controlpanel -> Shared folders
- Right click folder -> Edit
- Permissions
    - Give _guest_ read/write permissions
- NFS permissions
    - Add new entry
    - Use IP of pi
    - Read/write
    - Squash all users to guest
    - Check asynchronous
    - Check access to subfolders

Now, to allow the pi to change permissions etc., do the following:
- File station
- Right click folder -> Properties -> Permissions
- Give _guest_ administration permissions (Full control)

### Plex
[Instructions](https://pimylifeup.com/raspberry-pi-plex-server/)

