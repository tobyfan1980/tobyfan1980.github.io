---
layout: post
title:  "IPSec with Dynamic Routing"
date:   2017-09-19 11:00:00 -0800
categories:    Networking
tags:    IPsec
---

Route-based VPN grows more popular than policy based VPN, since it is more flexible. Instead of using policy to refer VPN tunnel, a VPN tunnel is indirectly referenced by a route that points to a specific tunnel interface. 
The tunnel interfaces (vti) on the two VPN end points can communicate with each other using certain routing protocols. Thus, any changes of networks on each side wiil be notified to the other side. We don't need to change policy configs and restart VPN end points. 

Bellow is an example of config VPN connections with dynamic routing protocol. 

### For ipsec connections, Libreswan is used. 

Below is ipsec.conf on one VPN endpoint.

```
config setup
        protostack=netkey
        logfile=/var/log/pluto.log
        # only enable plutodebug when asked by a developer
        plutodebug=control
        # Enable core dumps (might require system changes, like ulimit -C)
        dumpdir=/var/run/pluto/
        listen=10.0.149.110
include /etc/ipsec.d/*.conf
```

and tunnel1.conf in /etc/ipsec.d/

```
conn tunnel1
    left=10.0.149.110
    right=10.0.149.111
    leftsubnet=0.0.0.0/0
    rightsubnet=0.0.0.0/0
    auto=start
    authby=secret
    ikev2=no
    ike=aes256-sha512;modp4096
    phase2alg=aes256-sha512;modp4096
    dpddelay=30
    dpdtimeout=120
    mark=5/0xffffffff
    vti-interface=vti02
    vti-routing=no
    leftvti=192.168.10.3/24
```

It defines the vti and its ip. (The VPN endpoint on the other side has the similar config, vti ip=192.168.10.4)
The tag 'mark' refers to the ip route table for the vti traffic, if you have that table defined, you need to add route in it to forward the traffic in vti network to vti interface. 


### For routing, quagga is used
 
Quagga is a routing software suite implementing many routing protocols including OSPF, RIP, BGP, etc. 
In this example, we use RIP. So we edit /etc/quagga/daemons as below

```
zebra=yes
bgpd=no
ospfd=no
ospf6d=no
ripd=yes
ripngd=no
isisd=no
babeld=no
```

Then we config RIP in /etc/quagga/ripd.conf
```
! -*- rip -*-
!
! RIPd sample configuration file
!
! $Id: ripd.conf.sample,v 1.1 2002/12/13 20:15:30 paul Exp $
!
hostname ripd
password zebra
!
! debug rip events
! debug rip packet
!
router rip
 network 192.168.20.0/24
 network 192.168.10.0/24
 passive-interface eth1
!access-list private-only permit 10.0.0.0/8
!access-list private-only deny any
!
log file /var/log/quagga/ripd.log
!
log stdout
```

Here 192.168.10.0/24 is the network that VTIs are connected. 192.168.20.0/24 is the network on this side of VPN end point. 
eth1 is the NIC connected to 192.168.20.0/24, so it is set passive to not casting RIP packets.
 
### NOTE:
one problem worth to mention is that, in RIP, a router periodically  (default to 30 sec) multicast its routes to neighbors, 
however, the VTI interface by default has multicast disabled. As a result, a route from neighbor will not get updated and becomes expired. 
So, we need to enable multicast on the VTI interface. 
 
```
infonfic vti02 multicast
```

Then, using tcpdump on port 520, we can see the multicast packets like
```
18:33:59.106697 IP (tos 0xc0, ttl 1, id 43025, offset 0, flags [DF], proto UDP (17), length 52)
    192.168.10.4.520 > 224.0.0.9.520:
        RIPv2, Response, length: 24, routes: 1 or less
          AFI IPv4,    192.168.21.0/24, tag 0x0000, metric: 1, next-hop: self
18:34:01.672231 IP (tos 0xc0, ttl 1, id 46789, offset 0, flags [DF], proto UDP (17), length 52)
    192.168.10.3.520 > 224.0.0.9.520:
```

