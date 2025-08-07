---
title: "EqualLogic VMware iSCSI Setup -- Part 1: Preparation"
date: 2013-02-24T00:00:00Z
draft: false
tags: ["equallogic", "iscsi", "vmware", "san", "networking", "dell"]
categories: ["equallogic-vmware-iscsi-setup"]
author: "Harry John"
---

![Dell VMware](/images/dell-vmware1.png)

EqualLogic VMware iSCSI setup best practices have gone through many changes recently, fortunately Dell and VMware seem to have partnered well and made some good improvements towards the integration of their products. Development may well settle down now along with the release of ESXi 5.0/5.1 and Dell's own Multipathing Extension Module (MEM) available to those with Enterprise vSphere licencing.

There is a lot of documentation -- whitepapers, manuals and blog posts on the topic of iSCSI setup which can become overwhelming quite quickly. The setup is actually really simple with the correct up to date documentation to hand. The entire process of the SAN network installation can be broken down into the following steps:

1. SAN fabric design
2. Switch configuration
3. Array configuration
4. Host configuration
5. Monitoring

## EqualLogic VMware iSCSI Documentation

Firstly I do recommend you visit the [Rapid EqualLogic Configuration Portal](http://en.community.dell.com/techcenter/storage/w/wiki/3615.rapid-equallogic-configuration-portal-by-sis.aspx "Rapid EqualLogic Configuration Portal by SIS - Dell Storage Community") which is also broken down into 5 simple stages similar to how I have arranged this article. Download each document appropriate to your hardware and have a read through. EqualLogic have included sections where you can document your IP address settings, I re-created a lot of this into an Excel spreadsheet for my documentation and have published a copy for you fine people to use 🙂 I recommend you download the document and open it in Excel to make full use of the functions and formulas.

[SAN Network Configuration.xlsx](https://docs.google.com/file/d/0B8aXU-ZIYTeuOGs4NTVJUGUwODg/edit?usp=sharing "SAN Network Configuration.xlsx")

## iSCSI Network Settings

It is good practice to document to as much as possible before you start. This helps you during the planning process to know that you have everything clearly set out, whilst implementing you know you are entering the correct settings and if anything goes wrong you have a reference of what should be in place. In short the settings you need to prepare are as follows:

- Host iSCSI vmkernel IP addresses
- Host vmkernel NIC names
- EqualLogic group IP address
- EqualLogic interface IP addresses

In the next step of our EqualLogic VMware iSCSI setup we will look at the hardware and network fabric keeping a few important design considerations in mind.

**EqualLogic VMware iSCSI Setup -- Part 1: Preparation**  
[EqualLogic VMware iSCSI Setup -- Part 2: Network Configuration](/posts/equallogic-vmware-iscsi-setup-part-2-network/)  
[EqualLogic VMware iSCSI Setup -- Part 3: PowerConnect Configuration](/posts/equallogic-vmware-iscsi-setup-part-3-powerconnect-configuration/) 