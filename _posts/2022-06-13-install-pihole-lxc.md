---
layout: post
date: 2022-06-17
categories: [linux,services,lxc,proxmox]
tags: [server,linux,lxc,debian,pihole]
title:  "Install Pi-Hole DNS filter"
---

# Install Pi-Hole in a Proxmox container (LXC Template)

## Download the Debian template

To start you will need to have the CT template on proxmox, go to your proxmox node, select the storage (in my case 'local') and 'CT Templates'.
Here you have the change to 'Upload', 'Download from URL' or 'Templates'. You click 'Templates' and then search for 'Debian 11' (Or other distro of your preference).

## Create CT 

Top of the Proxmox GUI, 'Create CT'

Fill up the usual info

    - Choose a ID and Hostname
    - Uncheck 'Unprivileged container'
    - Create password

Choose your Template, and for hardware i gone for:

    - 4Gb Disk (nothing to see here) 
    - 2 Core (plenty)
    - 1Gb DDR (no need more)

Network configuration:

If you have VLANS and want all by protected by Pi-Hole you'll need to add them on the network section:
Only one need/can have gateway

    - eth0 IP / Netmask + Gateway = VLAN1
    - eth1 IP / Netmask = VLAN2
    - eth2 IP / Netmask = VLAN3
    - ....



## Install pi-Hole

SSH in to the container.
Start by updating the distro..

```bash
apt update && apt upgrade -y
```

Install Curl,

```bash
apt install curl -y 
```


Install Pi-Hole himself

```bash
curl -sSL https://install.pi-hole.net | bash
```

You can change admin password using,

```bash
pihole -a -p
```

## Notes

Intallation is straght forward.

- If you get error at the end that couldn't start services, <b>just</b> run the installation script again:

```bash
curl -sSL https://install.pi-hole.net | bash
```


- When installing, choose the main network (eth0) then you can listening to all networks (Permit all origins). 


More adblock & Malware list can be found here:

        https://filterlists.com/
