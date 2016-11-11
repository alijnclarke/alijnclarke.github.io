---
layout: post
title:  VMware Datastore.. speeds?
date:   2016-11-11 00:45
categories: [vmware]
---

#### Setup

So currently I’m using two Synology Diskstations as Datastores for my vSphere enviroment, the primary unit being a DS916+ optioned with 8GB of ram.

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

[image-1]:	/static/img/post-images/syno-storage.png
[image-2]:	/static/img/post-images/vmNFS01.png "vmNFS01 hosts *17 virtual machines*"
[image-3]:	/static/img/post-images/vmNFS02.png " vmNFS02 hosts *14 virtual machines*"