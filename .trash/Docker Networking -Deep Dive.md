# Docker Networking -Deep Dive. This is another post on my series on… | by Kasun Rajapakse | Ascentic Technology | Medium

Created: April 22, 2023 10:12 PM
Reviewed: No
URL: https://medium.com/ascentic-technology/docker-networking-deep-dive-bdeb39105bc

This is another post on my series on Docker. In this post, we will focus on docker networking and deep dive into how it works underneath. This post was written based on docker single-host networking.

## Docker Networking

Docker abstracts the underling networking of the host to provide the networking capability to the containers running. Each container attached to the docker network allocates a unique IP address that routable from container to container.

Following diagram shows how the docker network is working

Docker supports networking as first-class entities. Therefore docker network has its own life cycle and is not bound to other docker objects. We can manage and interact with networking by using `docker network` command.

Before we go into more details lets explore what are the existing networks are available in docker by default. To view existing docker networks we can use the command `docker network ls`

As can see in the above image by default docker provide three networks with the installation. The network `bridge` connected using bridge driver and it provides the intercontainer connectivity to all the containers running on the host. Most of the time when creating user define networks for containers we use `bridge driver` networks, other network types have their own usages you can find more detail on the below link.

## Create user define networks

To create a new network we can use the following command.

```
docker network create \
  --driver bridge \
  --attachable \
  --scope local \
  --subnet 10.0.42.0/24 \
  --ip-range 10.0.42.128/25 \
  example-network
```

- -driver — Network Driver use for the network
- -attachable — This allows us to attach or detach containers at any time
- -subnet — IP CIDR used for the network
- -ip-range — Allowed IP range for the containers

Next, run a container on newly created network to see the IP is allocating to container from the attached network.

```
docker run -it \
--network example-network \
--name network-explorer \
alpine:3.8 \
sh
```

- -network — If not specified attach the container to the default network.
- -name — Container Name

when using the command `ipaddr` inside the container to verify the IP address detail we can see the IP address is allocated from the specific network IP range we assigned.

In docker networking, we don't have any network policies or firewall. Therefore the containers within the network can talk to each other.

## Conclusion

We can use docker network for our application to isolate the container communications. With docker networking, we can use user define networks to customized based on the need.