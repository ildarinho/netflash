---
title: Load Balancing on HAproxy in EVPN
date: 2026-11-09 12:00:00 +0900
categories: [Networking, EVPN]
tags: [lb, frr, evpn, dc]
read_time: false
---
Load balancers are one part of a data center, used to distribute traffic for application purposes. The EVPN/VXLAN fabric is one of several options for the DC core, and it can be built in different ways depending on where VXLAN terminates. This post used the architecture where VXLAN terminates on the leaf, and Figure 1 and 2 shows the lab topology. The goal is for Peter to reach MJ and Ned through Manhattan. Files are available in my [git](https://github.com/ildarinho/netflash_files/tree/main/Load-Balancing-on-HAproxy-in-EVPN).

>The lab was built in EVE-NG using vMX running Junos 23.2R2, which supports MAC-VRF (introduced in 20.4R1). The lab is IPv4 only. If you don't have the opportunity to set it up, all configs, show command output, and packet captures are available in my git.
{: .prompt-info }

### Topology overview:

![Topology](/assets/img/posts/2.1.png)
_Figure 1_
![Topology](/assets/img/posts/2.2.png)
_Figure 2_

NY DC is built using an ERB overlay design with an IPv4 eBGP underlay where Leafs have their own unique ASN and all spines share the same ASN. Leaf links that are connected to Spine include in their own aggregated Ethernet interfaces that make our lab look more production style. NY has its own PI address pool and AS52, and also includes a load balancer named Manhattan.
Manhattan has connections to border leaf1, leaf 2 and ISP router via eBGP.  Load balancer works on ubuntu server 22.04 and used FRR for routing and haproxy for load balancer task with public ip address 52.0.0.52 that send to ISP via eBGP

Peter is a Ubuntu 22.04 desktop client connected to an ISP called Marvel within AS1.
MJ and Ned are two Ubuntu 22.04 servers, each running a small python script that returns info, which server responds. MJ is multihomed to Leaf1 and Leaf2 through an aggregated Ethernet interface, while Ned is connected to Leaf2.

![Topology](/assets/img/posts/2.3.png)
_Figure 3_

NY DC uses an ERB overlay design with symmetric IRB and a routed overlay with Type-5 routing, where the MAC-VRF is configured with the VLAN-aware service type. EVPN Type-2 MAC and MAC+IP handles intra reachability within a VLAN, and EVPN Type-5 handles inter reachability across IRB interfaces related to the VLAN.
Leaf1 and Leaf2 both have IRB interfaces related to VLAN 100 and VLAN 200.
Bleaf1 and Bleaf2 have only IRB interfaces for VLAN 100 and VLAN 200, and also have an interface ge-0/0/1 connected to the LB and assigned to the tenant-1 VRF.

The topology overview is finished. Here is the list of commands that I used to collect the information:
```bash
show lacp interfaces extensive
show interfaces
show interfaces ae100 extensive
show bridge domain
show bridge mac-table
show route
show route forwarding-table
show route table bgp.evpn.0 extensive
show mac-vrf forwarding instance extensive
show mac-vrf forwarding flood
show mac-vrf forwarding mac-table
show mac-vrf forwarding mac-ip-table
show mac-vrf forwarding vxlan-tunnel-end-point remote
show mac-vrf forwarding vxlan-tunnel-end-point esi
show policy __vrf-import-__default_evpn__-internal__
show policy __vrf-import-macvrf-1-internal__
show mac-vrf routing database
show bgp summary
show route forwarding-table
show system connections | match 179
show evpn database
show evpn database extensive
show evpn instance macvrf-1 extensive
```

### Load balancers
Manhattan is an Ubuntu server running FRR and HAproxy daemons

FRR conf:
```bash
conf here
```

HAproxy conf
```bash
conf here
```
> HAproxy work in mode with

A small python script is used on the MJ and Ned servers.
Small python script:
```bash
script here

```

### Test LB
The full flow is here:
![Test](/assets/img/posts/2.4.png)
_Figure 4_

Peter initiates a TCP connection with 52.0.0.52 and keeps it open for the purpose of sending HTTP requests and receiving responses.
Manhattan, running HAproxy, receives the HTTP request from Peter and writes it into the request buffer. Then HAproxy round-robin algorithm selects between MJ and Ned.
Manhattan establishes a TCP connection from 10.3.31.1 to Ned and sends the request from the request buffer. Ned response is received into the stream response buffer and then sent back to Peter over the original frontend connection.

Here is how the HAproxy stats look:
![Test](/assets/img/posts/2.5.png)
_Figure 5_

Let's be happy for Peter! After saving New York and Earth so many times, he deserves to stay in touch with friends.
