# dockerswarm
A Docker Swarm setup


## A three node docker swarm setup

Use either a VM or Physical machine and install Debian on the 3 machines.
In this documentation we assume you installed a user called `lucrasoft`.
The first node will act as both
- the manager node
- the storage node = add an additional virtual disk , as location for the mounts/volumes.


## Setup sudo for convenience

Debian lacks the sudo command by default. Let's add it.

``` bash
su -
apt install sudo
usermod -aG sudo lucrasoft
exit
su lucrasoft
```

## Install docker

We simply follow the docker documentation found at https://docs.docker.com/engine/install/debian/

A brief summary of the commands are listed [here](install-docker.md)

## Add hosts

In each node, add the nodenames and ip addresses, which can be found in `/etc/hosts`

```
sudo nano /etc/hosts

10.2.0.1	LS-Swarm01
10.2.0.2	LS-Swarm02
10.2.0.3	LS-Swarm03
```

## Add user to docker group

This helps to execute docker commands as a user.

`sudo usermod -aG docker lucrasoft`

## Initialize Swarm
On the manager node, we initialize the swarm.

```docker swarm init --advertise-addr 10.2.0.1```

And on the other nodes, we join the swarm with
`docker swarm join --token <token-as-displayed-during-swarm-init> 10.2.0.1:2377`

## Configure additional disk

Configure the additional disk on LS-Swarm01. We will use this disk as a NFS share for every node. 
Use fdisk to add a primary partition on the extra disk

```
fdisk 
(use option n to create new partition)
(choose primary partition)
(choose default values)
(use w to write)
```

Create the filesystem

```sudo mkfs -t ext4 /dev/sda1```

Check filesystem is correcy

```lsblk -f```

## Mount additional disk

To use the disk, we need to mount it.

```mkdir /mnt/datadisk```

We use blkid to identify the UUID of the new datadisk

```sudo blkid```

Add the mount to \etc\fstab to mount the disk permanently.

```sudo nano /etc/fstab```

Add this line

```UUID=<id> /mnt/datadisk   ext4  defaults  0 0```

## NFS Server

Install NFS Server on the manager node, with the extra disk.

```
sudo apt-get update
sudo apt-get install nfs-kernel-server -y
```




