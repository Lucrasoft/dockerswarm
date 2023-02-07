
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

## Core services

Two core services needs to be installed outside the Portainer GUI
- Portainer
- Traefik




