---
layout: post
title:  HomeLab Virtual Machine Datastores
date:   2016-11-11 00:45
categories: [vmware, homelab]
---

#### Intro
I’ve been running a home lab of some description for a little over a year, but only recently have I really ramped up both the amount of virtual machines running and the intensity of those tasks. 

However I’ve never done any assessment of how my storage equipment is dealing with it all.. 

Oh and I’m also trying to convince myself that there is a need for a proper business grade storage solution (thinking FreeNAS with a HBA card linked to an external DAS chassis) and 10Gb networking.
#### Setup

So currently I’m using two Synology Diskstations as Datastores for my vSphere environment, the primary unit being a DS916+ optioned with 8GB of ram.

The disks installed in this unit are as follows:
- 2x 3TB WD Red
- 2x 6TB WD Red
I won’t talk about the other Diskstation as that is very much a retired platform only used for temp vm’s.

The DS916+ (only) has two 1GB network interfaces, these are currently not aggregated and are both set (at switch level) to different VLANs.

- Volume 1 contains both 3TB Red’s and is setup as *SHR - 1 disk fault-tolerence* with a *ext4* filesystem. This has a usable capacity of *2.68TB*.

- Volume 2 contains the remaining two Reds and is setup as *SHR - 1 disk fault tolerance* and a *btrfs* filesystem. This has a usable capacity of *5.24TB*
![][image-1]

> Volume 1 is exposed to vSphere as *vmNFS01* and Volume 2 is exposed as *vmNFS02* (you guessed it, both over NFS v3).

vmNFS01 hosts *17 virtual machines*
![][image-2]
 vmNFS02 hosts *14 virtual machines*
![][image-3]


Although some vms have bene vMotioned from 1 \> 2, so I’m unsure as to why they are still being listed  and showing as active (any thoughts on this would be welcomed).

#### Active lab (aka production) tests
So I’m aware that this setup is absolutely not ideal, but currently it seems to be keeping up with my demands (granted none of my vms are IO intensive.. yet).

To start with I’m running an *iperf3* speed test from one of the virtual machines to the Disktation (DS916+, known as *syno01 from now on*). As this is a fairly high end SMB grade Diskstation it supports Synology’s Docker solution.. this allowed me to easily install *iperf3*.

I’m first of all going to run the speed test from a virtual machine who’s hard disk is provided by *vmNFS02* (hosted on this nas).
![][image-4]
This gives a rather disappointing result of *368 Mbits/sec*, but we have to remember that this is with the box hosting the datastore for *31* running vms.

So for argument’s sake, here’s the same test ran on a vm which is running on directly attached storage. 
![][image-5]
Not much difference hey, I guess all those vms are using their fair share of bandwidth!

Before we start shutting vms down there’s one last thing to try, remember I said that *syno01* has two nics? Well one of those is not used at all for vm NFS traffic and is exclusively used for lan user smb and backup traffic.. bingo, let’s try that.
![][image-6]
**wow** that’s more like it, so looks like it was the vm traffic saturating the line. It’s still not quite up to the 125 Mb/s you’d expect (at least to peak to), but I’m pretty sure that my IP cameras use that NIC.. so that may explain the missing bandwidth (synology make it very tricky to figure out which applications are using which interface..).

#### Nuke all the vms!
So I’ve vMotioned or powered down any virtual machine operating on either *vmNFS01* or *vmNFS02* (any datastore on *syno01*.
I’ve also temporarily disabled other file sharing or bandwidth intensive tasks (hosting my ip camera system for one), just to make sure there is as little pipe saturation as possible.

##### Let’s re-run the tests

[image-1]:	/static/img/post-images/syno-storage.png
[image-2]:	/static/img/post-images/vmNFS01.png "vmNFS01 hosts *17 virtual machines*"
[image-3]:	/static/img/post-images/vmNFS02.png " vmNFS02 hosts *14 virtual machines*"
[image-4]:	/static/img/post-images/iperf3-1.png
[image-5]:	/static/img/post-images/iperf3-2.png
[image-6]:	/static/img/post-images/iperf3-3.png