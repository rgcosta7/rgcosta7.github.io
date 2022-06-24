---
layout: post
date: 2022-06-17
categories: [homelab,docker,vm]
tags: [server,proxmox,vm,virtual machines,docker,debian]
title:  "Install Docker in a proxmox VM!"
---

# Install Docker in Debian 11 VM

## **Install Debian**

How-to? WIP

## **Install Docker and docker-compose**

Always update the system:
```bash
sudo apt update && sudo apt upgrade
```

Install needed packages.
```bash
sudo apt install -y \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    gnupg \
	    lsb-release
```

Get key to authenticate docker repo.
```bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

```

Add docker repo.
```bash
sudo echo \
  		"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  		$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Run repo update
```bash
sudo apt update
```

Install docker.
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Check docker is running.
```bash
sudo systemctl status docker
```

Allow docker to run  without root privileges.
```bash
sudo usermod -aG docker $USER
```

Log Out then Back In and test docker
```bash
docker version
```

## **Install docker-compose**

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.5.1/docker-compose-linux-$(uname -m)" -o /usr/local/bin/docker-compose 

sudo chmod +x /usr/local/bin/docker-compose 

docker-compose -v
```


## **Enable SMB shares for the media files**

Download the package cifs:
```bash
sudo apt install cifs-utils -y
```

Store the user credencial on root:
```bash
nano /root/.smbcredentials

	username=username
	password=password
```

Edit fstab to add mount point to the boot:
```bash
sudo nano /etc/fstab
```

Add line to fstab:
```bash
//10.8.0.188/media /media/media cifs uid=1000,gid=1000,credentials=/root/.smbcredentials
```

Mount the shared folder
```bash
mount -a -vvv
```

## **Install local-persist plugin for docker 
```bash
		curl -fsSL https://raw.githubusercontent.com/MatchbookLab/local-persist/master/scripts/install.sh | sudo bash
```

## Mounting NFS share

Install NFS
```bash
apt -y install nfs-common
```
Create mount directory
```bash
 mkdir /shares
```
Test mount the shared folder,
```bash
mount -t nfs 10.10.0.10:/backups /var/backups
```
Add the mount to the boot, edit ``` /etc/fstab ```
```bash
10.10.0.10:/backups /var/backups  nfs rw
```
If not mounted before you can use the follow command:
```bash
mount -a -vvv		
```
