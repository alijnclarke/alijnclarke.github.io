---
layout: post
title:  
date:   2016-11-29 21:45
categories: [vmware, homelab, vsphere]
---
#### The problem
Recently I’ve been evacuating all of my virtual machines from one NFS Datastore (*vmNFS01*) to another (*vm_pool*), most of my 30 vm’s vMotioned successfully, but some left behind some entries in *vmNFS01*’s vm association table. This was odd behaviour that I hadn’t seen before, so I did a bit of digging.

As *vmNFS01* is a Synology NAS, I was able to browse to the directory and look to make sure vMotion had indeed moved the files, it had and there was nothing there. Just to be sure I opened up *vm_pool* (Freenas based) and indeed the vm files were there.

**weird**

#### The fix
After a bit of googling I found an article detailing a similar problem on the vmware website (kb 2105343)
> [\[]https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2105343]()

Basically you need to firstly make sure that non of your vm’s have an iso mounted from the datastore, this was okay in my case. But heres the killer… you need to make sure that in any snapshots you’ve also not got an iso mounted. This was my problem as with each vm I had created a ‘fresh install’ snapshot that did indeed have the iso from *vmNFS01* mounted. To fix it I simply removed all the snapshots from each vm, although I do believe that you can simply remove the culprit snapshot and that will solve your problem.
> Be aware that removing snapshots can take a significant amount of time.

This post was more for my reference, but I hope it helps someone.

