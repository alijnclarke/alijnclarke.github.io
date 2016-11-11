---
layout: post
title:  HomeLab Virtual Machine Datastores
date:   2016-11-11 00:45
categories: [vmware, homelab]
---

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
![][image-5]
So for argument’s sake, here’s the same test ran on a vm which is running on directly attached storage. Not much of a difference eh?.. I guess those running vms must be taking their toll!

#### Nuke all the vms!
So I’ve vMotioned or powered down any virtual machine operating on either *vmNFS01* or *vmNFS02* (any datastore on *syno01*.
I’ve also temporarily disabled other file sharing or bandwidth intensive tasks (hosting my ip camera system for one), just to make sure there is as little pipe saturation as possible.

### Let’s re-run the tests

[image-1]:	/static/img/post-images/syno-storage.png
[image-2]:	/static/img/post-images/vmNFS01.png "vmNFS01 hosts *17 virtual machines*"
[image-3]:	/static/img/post-images/vmNFS02.png " vmNFS02 hosts *14 virtual machines*"
[image-4]:	/static/img/post-images/iperf3-1.png
[image-5]:	/static/img/post-images/iperf3-2.png