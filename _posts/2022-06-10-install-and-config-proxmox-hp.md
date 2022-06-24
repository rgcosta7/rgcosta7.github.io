---
layout: post
date: 2022-06-10
categories: [linux,proxmox,lxc]
tags: [server,linux,lxc,container,debian,plex,proxmox,media]
title:  "Install and config Proxmox 7.2 on a HP server"
---


# Install and config Proxmox 7.2 on a HP server

## Download the ISO

You can find on the official website, [Download ISO](https://www.proxmox.com/en/downloads/category/iso-images-pve)

## Install Proxmox

Easy installation, make sure you select the right hard-drive and put the right configuration for your network. later you can configure <b>VLANS</b> 

When finish install, the machine will reboot and will be ready.
You'll see the URL on the screen:
```
https://192.168.70.180:8006
```


## Configurations
 
Most of this configuration can be done on Proxmox GUI but here I'll use the CLI.



### Repositories:

Add no subscription proxmox repo:

```bash
nano /etc/apt/sources.list
```

Add to the end:

```bash
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
```

You can remove/comment  the enterprise repo,

```bash
nano /etc/apt/sources.list.d/pve-enterprise.list
```

Make sure you have all updated packages:

```bash
apt update && apt upgrade -y && apt dist-upgrade -y
```

### Patch Kernel and modules for PCI passthrough  (HP SERVERS):

In my case I use a DL380 G9 and couldn't pass my 4 NIC PCI to the pfsense, I need to change the kernel to alow it. I found this GitHub page from [MichaelTrip](https://github.com/MichaelTrip/relax-intel-rmrr), a big thanks to him!!

#### 1 - Download the patched kernel

At the shell:

```bash
mkdir iommu && cd iommu
```

and download the patched kernel:

```bash
wget https://github.com/MichaelTrip/relax-intel-rmrr/releases/download/v1.0.8/linux-tools-5.13-dbgsym_5.13.19-4_amd64.deb
wget https://github.com/MichaelTrip/relax-intel-rmrr/releases/download/v1.0.8/linux-tools-5.13_5.13.19-4_amd64.deb
wget https://github.com/MichaelTrip/relax-intel-rmrr/releases/download/v1.0.8/pve-headers-5.13.19-2-pve-relaxablermrr_5.13.19-4_amd64.deb
wget https://github.com/MichaelTrip/relax-intel-rmrr/releases/download/v1.0.8/pve-kernel-5.13.19-2-pve-relaxablermrr_5.13.19-4_amd64.deb
wget https://github.com/MichaelTrip/relax-intel-rmrr/releases/download/v1.0.8/pve-kernel-libc-dev_5.13.19-4_amd64.deb
```

Install the kernek,

```bash
dpkg -i *.deb
```

#### 2 - Set the kernel as default

Find the new kernel IDs,

```bash
grep menu /boot/grub/grub.cfg
```


From the previus output you will need to find the submenu option ID, something like this ``gnulinux-advanced-83ea5c69-120d-4ba6-8d91-e3de2dc98b0a`` and the relaxablermrr ``gnulinux-5.13.19-2-pve-relaxablermrr-advanced-83ea5c69-120d-4ba6-8d91-e3de2dc98b0a``

Copy both from your output, you'll need it to set them as a default kernel in each boot.


Edit grub,

```bash
nano /etc/default/grub
```

On the ``GRUB_DEFAULT`` you need to paste the IDs that you copy early in this format:

```bash
GRUB_DEFAULT="SUBMENU_ID>MENUENTRY_ID"
```


You'll need to add ``intel_iommu=on,relax_rmrr iommu=pt`` to the line ``GRUB_CMDLINE_LINUX_DEFAULT="quiet"`` to look like this,

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on,relax_rmrr iommu=pt"
```

#### 3 - Load the neccessary modules  

Edit ``/etc/modules`` and add to the end:

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

And to finish, one last option need to be add:

```bash
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
```

Now you can reboot, finger crossed that all was done right,

```bash
shutdown -r now
```

After boot you can confir wich kernel you using by typing,

```bash
uname -r
```

You should get  ``5.13.19-2-pve-relaxablermrr`` at the time I was writing this page.
And you can check if IOMMU was loaded,

```bash
dmesg | grep 'Intel-IOMMU'
```

### Expand local volume (merge all the volumes) 

On the proxmox webgui go to ``Datacenter --> Storage`` select local-lvm and remove it!

Go to proxmox shell and,

```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root  
```

This should it, you can see that now you just have local and all the free space is in local.

Again, go to ``Datacenter --> Storage`` and edit local to be use for all the purpose, 'Disk image', 'ISO image'...all of them in my case!


### Container Templates

If is missing some templates from the list, special the TurnKey you'll need to run the follow command on the shell:

```bash
pveam update
```