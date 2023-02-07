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
```

## Install docker.

See install-docker.md

## Add user to docker group

This helps to execute docker commands as a user.

`sudo usermod -aG docker lucrasoft`


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

`timedatectl` 
make sure timezone is correct!

## Let worker node join the swarm!

on the manager node, ask for the join token by:
`docker swarm join-token worker`

no the worker node, join the swarm with:

`docker swarm join --token <token> <managernode-ipadres>:2377`






