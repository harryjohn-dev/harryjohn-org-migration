---
title: "EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration"
date: 2013-02-26T00:00:00Z
draft: false
tags: ["equallogic", "iscsi", "vmware", "san", "networking", "dell", "powerconnect"]
categories: ["equallogic-vmware-iscsi-setup"]
author: "Harry John"
comments: true
---

![SAN LAN Network](/images/SAN-LAN-Network-e1361849167569.png)

The EqualLogic iSCSI network consists of the switches, EqualLogic arrays and ESX hosts, all of which need configuring properly to make the most of each others load balancing, performance and availability features. In terms of the network fabric, there are a few things we need to consider:

1. High availability -- multiple links and switches to provide redundancy
2. Performance -- elimination of bottlenecks
3. Management -- monitoring and management network

This guide can apply to a range of hardware as long as we are using EqualLogic arrays and VMware ESX/ESXi hosts. The switches can be Dell or Cisco for example, as long as they are on the Dell Approved Hardware list. The hardware I used when making this guide is as follows:

- 2 x PowerConnect 7048
- 6 x PowerEdge R710
- 2 x EqualLogic PS6000

Here is some brief detail about my hardware specifications, and how I put the key design considerations mentioned earlier into practice when configuring each component.

## EqualLogic PS6000 array members

*4 GbE ports/ controller*  
*2 controllers/member (active passive)*

I will mention management first as the PS6000 does not have a dedicated management port. I *could* dedicate one of the four Ethernet ports to management only but this would leave me with three links for iSCSI -- two to switch A but **only one link to the other switch B**. In the case of a switch failure, we could have **33% available iSCSI bandwidth compared to normal conditions**.

For this reason I opted for all four ethernet ports to serve both iSCSI and management for the members, I will configure a VM with SAN HQ on the iSCSI network for monitoring and email reporting.

## PowerConnect 7048 iSCSI switches

*Stacking modules*

Two switches in a stack will keep the network alive in the case of a switch PSU or complete failure -- you could also add redundant PSU to the switches for extra protection. With the stacking modules we have a fast Inter-Switch Link (ISL) -- whether you LAG or stack you need to make sure this link is fast as iSCSI traffic can and will traverse the ISL. I will mention the pros and cons of LAG vs stack in the switch configuration part of this guide.

The 7048 switches have a high port buffer memory and are approved EqualLogic iSCSI switches, flow control and all the other iSCSI goodies are available. These switches also have an Out-of-Band management port each and these will be connected to our LAN network to give us management access to the switches.

## VMware ESXi 5.1 hosts

*Dell PowerEdge R710*  
*2 GbE NICs for iSCSI data*

[Two NICs per host is a **requirement** for multipathing to work properly and allow the hosts and EqualLogics to load balance. You can add more NICs in pairs if your network utilisation is high.]

Now we have our EqualLogic VMware iSCSI network designed to best practice we can move on to look at the PowerConnect switch configuration.

[EqualLogic VMware iSCSI Setup -- Part 1: Preparation](/posts/equallogic-vmware-iscsi-setup-part-1-preparation/)  
**EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration**  
[EqualLogic VMware iSCSI Setup -- Part 3: PowerConnect Configuration](/posts/equallogic-vmware-iscsi-setup-part-3-powerconnect-configuration/) 