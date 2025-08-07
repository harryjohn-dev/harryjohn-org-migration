---
title: "EqualLogic VMware iSCSI Setup -- Part 3: PowerConnect Configuration"
date: 2013-12-30T00:00:00Z
draft: false
tags: ["equallogic", "iscsi", "vmware", "san", "networking", "dell", "powerconnect", "switches"]
categories: ["equallogic-vmware-iscsi-setup"]
author: "Harry John"
---

![PowerConnect 7048 Switch](/images/34017.jpg)

So, recently I configured a set of PowerConnect 7048 switches for iSCSI following all the standards and best practices I could find. Actually I had prepared the PowerConnect iSCSI configuration a few months ago, I have been waiting for the Dell experts to confirm the configuration is current best practice and fit for our environment.

Configuring any PowerConnect model is a very similar process, however you must check the switch documentation you downloaded from the portal on part 1 of this guide -- there may be a few minor differences. I will note now that PowerConnect 5524/5548 switches are not approved for use in an iSCSI environment and that iSCSI Optimisation should actually be disabled on these switches.

If you are using a non-PowerConnect set of switches, there are a list of switch requirements within the [Dell EqualLogic Configuration Guide as of v14.3](http://en.community.dell.com/techcenter/storage/w/wiki/2639.equallogic-configuration-guide.aspx) -- this document does get updated frequently (on a quarterly basis to be precise) and has been much improved recently. I highly recommend reviewing this document fully as most of the information you need is now well documented here.

## iSCSI Setup Best Practices

**My goals when configuring these switchers were to:**

1. have a completely isolated iSCSI network
2. follow all best practices where possible
3. use the Out-of-Band (OOB) interface for switch management
4. add a new VMware VM Network port group to the iSCSI virtual network and use this for the SAN HQ server monitoring and management

**Also, the switch features we need to enable for iSCSI best practice are as follows:**

- Flow control
- Jumbo frames
- Portfast
- Storm control
- Non default VLAN

## EqualLogic SAN Infrastructure Best Practices

The following excerpt is from the [Dell EqualLogic Configuration Guide v14.3](https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&ved=0CEkQFjAB&url=http%3A%2F%2Fen.community.dell.com%2Fdell-groups%2Fdtcmedia%2Fm%2Fmediagallery%2F19852516%2Fdownload.aspx&ei=CojBUvHVNoiohAe2v4DQCw&usg=AFQjCNFmLxa1fyg3c9SbaryjGUnuJR6nYA&bvm=bv.58187178,d.ZG4), I have picked a few of the most important/least obvious best practices however I would still recommended you read through the entire list:

- It is strongly recommended that a physically separated network be used for iSCSI traffic and that this network not be shared with other traffic types.
- Rapid Spanning Tree Protocol must be enabled if the SAN infrastructure has more than two switches in a non-stacked configuration, and portfast must be enabled on all edge device ports (hosts, FS Series appliances and arrays).
- All switches within the SAN must be interconnected such that there is always a path from any Ethernet port on one array to all other Ethernet ports on all other arrays in the group.
- All switches and host network controllers within the infrastructure must have (at a minimum, receive) flow control enabled for optimal performance.
- Take advantage of your switch's VLAN capabilities. You may optionally create a VLAN dedicated to iSCSI traffic (even on dedicated switches). If necessary, create a second VLAN for management traffic. The actual VLAN configuration of your iSCSI SAN will be dictated by your SAN network design requirements and the features of the iSCSI SAN switches being used.
- Jumbo frames should be enabled for best performance. If you choose to use jumbo frames then all nodes in the SAN fabric must have jumbo frames enabled.
- For best performance and reliability, we recommend that all interconnection paths between non-stacking switches (LAGs) use a dynamic link aggregation protocol such as LACP

## PowerConnect 7048 iSCSI Configuration

Below are the exact configuration steps I took to install my PowerConnect 7048s. Please note all environments vary -- you should refer to the documents mentioned in the preparation section of this guide and ensure you follow the appropriate steps for your specific hardware.

**Step 1:** Enter configuration mode  
`enable`  
`configure`

**Step 2:** Configure management network  
`interface out-of-band`  
`ip address 172.16.0.101`  
`exit`

**Step 3:** Configure management security  
`line telnet`  
`login authentication default`  
`password [password]`  
`exit`  
`ip http authentication local`  
`username admin password [password] privilege 15`

**Step 4:** Enable iSCSI Optimisation and flow control  
`iscsi enable`  
`flowcontrol`

**Step 5:** Add iSCSI VLAN  
`vlan database`  
`vlan 100`  
`exit`  
`interface vlan 100`  
`name iSCSI`  
`exit`

**Step 6:** Enable jumbo frames, Portfast and storm control  
`interface range Gigabitethernet all`  
`switchport access vlan 100`  
`mtu 9216`  
`spanning-tree portfast`  
`no storm-control unicast`  
`storm-control multicast`  
`storm-control broadcast`  
`exit`  
`exit`

**Step 7:** Save config and reboot  
`copy running-config startup-config`  
`reload`

## PowerConnect iSCSI Stack vs LAG

Whilst multi-pathing will manage the paths between the hosts and arrays without an Inter-Switch Link (ISL), it is important that there is sufficient inter-switch bandwidth as the EqualLogic arrays are quite chatty and can produce a lot of network traffic between themselves, especially when performance and capacity load balancing are enabled with more than one array. As mentioned above in the list of best practices, all the switches in the SAN must interconnect.

The real question is whether you choose to interconnect by means of stacking or creating a Link Aggregate Group (LAG). There are some pros and cons of both:

### Stacking

**Advantages:**
- Simpler to implement
- Easier to manage multiple switches as a single switch
- Possibly higher bandwidth and lower latency than using link aggregation and Ethernet

**Disadvantages:**
- Cannot interconnect switches from different vendors
- Increases cost of switch
- Must schedule downtime when upgrading firmware as you cannot upgrade on each switch independently

### LAG

**Advantages:**
- Can interconnect switches from different vendors
- Cheaper cost of switch
- Can upgrade switch firmware on each switch independently -- no maintenance downtime

**Disadvantages:**
- Some solutions limited to eight port link aggregation group (8Gbps)
- Spanning tree protocol must be used if more than two switches are used -- causes some links to be "blocked" and reduces bandwidth availability
- Possibly less bandwidth than stacking
- More complex to implement

[EqualLogic VMware iSCSI Setup -- Part 1: Preparation](/posts/equallogic-vmware-iscsi-setup-part-1-preparation/)  
[EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration](/posts/equallogic-vmware-iscsi-setup-part-2-network/)  
**EqualLogic VMware iSCSI Setup -- Part 3: PowerConnect Configuration** 