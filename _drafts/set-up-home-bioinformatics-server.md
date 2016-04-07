---
layout: post
title: Hot to set-up a home bioinformatics server (old laptop)
date: 2016-02-14 19:02:00.000000000 +01:00
categories: []
tags:
- R
- Bioconductor
- Fisher
status: publish
type: post
published: true
---

WARNING: it should go without saying, but I will say it nonetheless. If you value anything in your current set-up, stop now. If you continue do a back-up before starting. Better yet. When was the last time you backed-up your files? Do it again now regardless.  


# Download iso
The iso can be found in [debian-8.3.0-amd64-netinst.iso](https://www.debian.org/CD/netinst/), and I choose the [latest (minimal) version](http://cdimage.debian.org/debian-cd/8.3.0/amd64/iso-cd/debian-8.3.0-amd64-netinst.iso)

During the installation there where issues with proprietary drivers, so I had to donwload a [version](https://www.howtoforge.com/perfect-server-debian-wheezy-apache2-bind-dovecot-ispconfig-3) that includes these. Otherwise no modem drivers. 

Edit: forget Debian, It doesn't have the broadcom drivers in the iso, so no internet. No internet, no distro.

## Ubuntu
[Download](http://www.ubuntu.com/download/server)

# create liveUSB

I used this [tutoriall](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-ubuntu)  because I am using Ubuntu. Unibotin is also a good option for other OS.

Also failed. So I turned to a light distro: [Xubuntu](http://xubuntu.org/getxubuntu/#lts) and [downloaded](http://ftp.uni-kl.de/pub/linux/ubuntu-dvd/xubuntu/releases/14.04/release/xubuntu-14.04.4-desktop-amd64.iso) version 14.04.4. 

Installation went smoothly and I could set-up wireless while installing. Ethernet still not working but that could an issue of home network.

## GUI
The linux version I ended up installing comes with a graphical interface, that whilst taking up disk space (not relevant in my setup), might come in handy if I need to use the computer as backup to my current laptop. That said, the GUI is not needed for the server and it is using resources. To [turn it off](http://askubuntu.com/questions/148321/how-do-i-stop-gui) use `CTRL + ALT + F1` to enter command-line mode, and `sudo stop lightdm`. Congratulations, you are now in command-line mode. `sudo start lightdm` or `startx` will bring it back up again. With the later once you log out of your account it reverts back to the command-line.

## ssh server

`sudo apt-get install openssh-server`, if needed start the ssh with ` sudo service ssh start`. Since connecting as sudo seems to be not advised for security reasons, I will create a users group:
`sudo addgroup work`
and a non-admin user: 
`sudo adduser adomingu`
This will be an acocunt with which I will do my work:
`sudo adduser adomingu work`

## static ip
`sudo nano /etc/network/interfaces`
> auto lo
> iface lo inet loopback
> 
> # static
> auto wlan0
> iface wlan0 inet static
>    address 192.168.0.11
>    netmask 255.255.255.0
>    network 192.168.0.1
>    gateway 192.168.0.0
>    broadcast 192.168.0.255
>    wpa-ssid wifi_name
>    wpa-psk wifi_password

## connect to the server:
[Tutorial](http://www.cellbiol.com/bioinformatics_web_development/doku.php/chapter_2_-_the_linux_operating_system/installing_and_using_openssh_server_for_remote_connections#installing_openssh_server)


# other neat things

htop, a CLI task manager that I really like
lm-sensors, a package that provides the option of reading all of the sensors on your motherboard
hddtemp, reads the hard disk temperature from S.M.A.R.T.
hdparm, which allows you to put hard disks in standby
tmux, a virtual console manager

This can be done in one command:
`sudo apt-get install htop lm-sensors hdparm tmux`


# Install Debian from USB
To boot from the USB press F12 when the computer starts, and choose USB in the boot menu options. Then I followed the instructions [here](https://www.howtoforge.com/perfect-server-debian-wheezy-apache2-bind-dovecot-ispconfig-3) (graphical install). I choose English (UK) as the language, Germany as a location, and Portuguese keyboard layout (yes, that is weird).

Hostname: pipette
username: antonio (Antonio Domingues)
password:
No encryption
No LVM
OpenSSH


# network configuration
https://help.ubuntu.com/community/NetworkConfigurationCommandLine/Automatic

# graphical interface
[Xfce](http://docs.xfce.org/xfce/getting-started) is a lightweight interface useful for under-powered and old computers. Install with:

```bash
sudo apt-get update &&
sudo apt-get install -y xfce4 xfce4-goodies
```

To start: `sudo startxfce4`. `sudo` is not required but it might useful to mount external drives. Exit by logging out.


# [reboot from command-line](http://askubuntu.com/questions/187071/how-do-i-restart-shutdown-from-a-terminal)
`sudo reboot`
