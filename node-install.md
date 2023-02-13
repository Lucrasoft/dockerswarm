# Per node (WORKER AND MANAGER) installation

## Setup sudo for convenience
Debian lacks the sudo command by default. Let's add it.

``` bash
su -
apt install sudo
usermod -aG sudo lucrasoft
exit
su lucrasoft
```

## Add all other hosts 

In each node, add the nodenames and ip addresses, which can be found in `/etc/hosts` 

```
sudo nano /etc/hosts

10.2.0.1	LS-Swarm01
10.2.0.2	LS-Swarm02
10.2.0.3	LS-Swarm03   
#etc. 
```

## Install docker.

See install-docker.md

## Add user to docker group

This helps to execute docker commands as a user.

`sudo usermod -aG docker lucrasoft`

## Gluster setup (on the WORKER nodes)

All nodes have an additional virtual disk attached. The set of these disk will form a clustered volume based on GlusterFS. Most commands come directly from their manual.

Identify the extra storage disk:
`fdisk --list`

Partition the extra disk with:
`fdisk /dev/sdb`

Format the partition (we use the builtin ext4 instead of the xfs)
`mkfs.ext4 /dev/sdb1`

Get the UUID 
`blkid`

Prep mount point
`mkdir -p /mnt/glusterdisk`

And add the following line to the /etc/fstab: `UUID=<YOURID> /mnt/glusterdisk ext4 defaults 0 0` with `sudo nano /etc/fstab`.

Test the mount with `mount -a`.

And install the gluster tooling (deprecated!)
`sudo apt install glusterfs-server`
The bundled packaged version of glusterfs on debian is (at the time of writing) v9.2 , but v10.x is already out. 

Updated -> Use the official gluster release as described on the gluster page!


## Gluster setup (on the MANAGER node)

The disk on the manager node is split in two partitions. The mount points are /mnt/witness1 and /mnt/witness2 (see /etc/fstab)

We use the witness disks as arbiter bricks, as described. We need two of them because GlusterFS wants a strict multiple of 3 bricks in the arbiter setup. That is : 2 replicat +1 arbiter.

To balance between redundancy and capacity, we created a volume with a 2x (2+1) setup.

The ordering of the brick-per-server is IMPORTANT! as it indicates which brick is a replica and which one is a arbiter.

`sudo gluster volume create glusvol1 replica 2 arbiter 1 ls-swarm02:/mnt/glusterdisk/brick ls-swarm03:/mnt/glusterdisk/brick ls-swarm01:/mnt/witness1/brick ls-swarm04:/mnt/glusterdisk/brick ls-swarm05:/mnt/glusterdisk/brick ls-swarm01:/mnt/witness2/brick force`

usefull commands:
- `gluster volume info`

Which outputs 

```
Number of Bricks: 2 x (2 + 1) = 6
Transport-type: tcp
Bricks:
Brick1: ls-swarm02:/mnt/glusterdisk/brick
Brick2: ls-swarm03:/mnt/glusterdisk/brick
Brick3: ls-swarm01:/mnt/witness1/brick (arbiter)
Brick4: ls-swarm04:/mnt/glusterdisk/brick
Brick5: ls-swarm05:/mnt/glusterdisk/brick
Brick6: ls-swarm01:/mnt/witness2/brick (arbiter)
```


## Gluster client-side (WORKER nodes only) 

prep a mounting point:
`sudo mkdir /mnt/volume1`

and change fstab to 
`localhost:/glusvol1 /mnt/volume1 glusterfs defaults,_netdev 0 0`



## NFS Client (WORKER ONLY)

Attach the shared datadisk of the manager node to this worn 
sudo apt install nfs-common

sudo mkdir /shared

Try to mount the nfs share manually first:
`sudo mount ls-swarm01:/mnt/datadisk/shared /shared`
and unmount again 
`sudo umount /shared`

Add a permanent mount in \etc\fstab
`sudo nano /etc/fstab`
add the line
`ls-swarm01:/mnt/datadisk/shared /shared nfs defaults 0 0`


# Recommended node settings 

Some dockers require specific host settings, especially all dockers having something todo with ElasticSearch. 

in `/etc/sysctl.d` folder, make sure to create a `10-es.conf` file with the following line:
`vm.max_map_count=262144` 



## Turn of swap 

## Check timezone!

Make sure the timezone is correct!
`timedatectl` 

## Let worker node join the swarm!

On the manager node, ask for the join token by:
`docker swarm join-token worker`

On the worker node, join the swarm with:
`docker swarm join --token <token> <managernode-ipadres>:2377`






