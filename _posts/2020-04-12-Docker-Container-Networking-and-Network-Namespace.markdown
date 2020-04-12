---
layout: post
title:  "My Notes: Docker Container Networking and Network Namespace"
date:   2020-04-12 01:00 +0300
published: false
categories: 
---

Let's have a look at container networking.

> These are my notes for container networking which I take while reading and studying container networking. See resources at the bottom to have a loook in detail about this topic.

## Container Namespaces 

Docker (or any container technology) uses linux network namespaces to isolate container network from host network. When Docker creates a container; it also creates a related network namespace for that container.

Then, Docker connects the new container with **docker0** bridge using a veth pair.  
A veth pair is like a network cable which has an network interface on each side.  
Thanks to that, the new container is connnected to the host's networking stack.  
(But of course some rules apply depending on the bridge type.)

![](https://res.cloudinary.com/safakulusoy/image/upload/c_limit,w_600/v1586644745/safakulusoy.com/container-networking/docker-container-network.jpg)

## VETH Pair 

Veth pair is the glue that connects the container network to the host network. 

Lets examine a veth pair to understand its role more.

When you run `ip addr` or `ip a` shortly on the host, you will see a list of network interfaces.  
All items start with a line number like `2: eth0` or`3: docker0` or `9: veth06b4106@if8`.  
Here you see the veth pair's network interfaces, but for only the host side.  
We need to run the same command in the container to see veth pair's container side network interface.  
Line numbers are important; we will use them in our investigations. 

First, Lets run `ip a` command on the host, filter for veth pair.
```
ubuntu@vm0:~$ ip a | grep 'veth.*if6'
7: veth926be0c@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP g
roup default
```

Then, Lets run `ip a` command on the first container, filter for veth pair:
```
ubuntu@vm0:~$ docker exec -it con1 ip a | grep eth0
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
```

Let's interpret the results.  

Take the first container as an example.  
First container's network interface's line number and name is: `6: eth0@if7`.  
And it connects to the counterpart `7: vnetxx@if6` network interface on the `docker0` bridge.

Please look carefully at the last part of the veth network interface names after the line number. i.e. "6: eth0@**if7**". 

Voilá. You can see that, veth pair network interface names end with the corresponding veth pair network interface's line number!


## Resources
- [Container Namespaces – Deep Dive into Container Networking (Platform9)](https://platform9.com/blog/container-namespaces-deep-dive-container-networking/)