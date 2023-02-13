
## Initialize Swarm
On the manager node, we initialize the swarm.

```docker swarm init --advertise-addr 10.2.0.1```

And on the other nodes, we join the swarm with
`docker swarm join --token <token-as-displayed-during-swarm-init> 10.2.0.1:2377`


## Core services

Two core services needs to be installed outside the Portainer UI
- Portainer itself
- Traefik

These two stack details are listed in this repo in the corresponding subfolders.


usefull commands you need to deploy a stack manually.

`docker stack deploy -c traefik.yml traefik`

`docker stack rm <stackname>`






