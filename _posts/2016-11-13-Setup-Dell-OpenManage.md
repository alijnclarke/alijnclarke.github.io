---
layout: post
title:  Setting up Dell OpenManage with vSphere integration
date:   2016-11-13 15:30
categories: [vmware, homelab]
---

#### Why and what does it do?
So with virtualisation becoming so popular, it makes sense to get a ‘single pain of glass’ view into both your virtual and bare metal appliances. OpenManage has been around for a while as a server application that allowed you to perform what was basically an enhanced iDRAC feature set (Dell’s built in server management system).

Recently however OpenManage has received an update which is of particular use to me, as my homelab runs vSphere 6U2. Basically you now get full integration in the vSphere web client (only the normal web client however, I’m really hoping that it’s updated to support the HTML 5 one soon!).

This means you get full hardware status updates, health reports and most importantly **updates**, yes.. instead of the long drawn out process that we all know of updating Dell servers, you can now do it directly from the vSphere console.. Awesome right!

#### Requirements
So to setup OpenManage for vSphere you’ll need the following:
- The OpenManage web server (runs on a vm, provides a web ui for sexiest agents - doesn’t need to run 24/7)
- A VIB which we deploy to each ESXI host
- vCenter integration ova

Obviously you’ll also need the bare metal hypervisors, in this case i’ll be adding two Dell PowerEdge servers:
- Dell PowerEdge R610
- Dell PowerEdge R710

#### Setup
I’m not going to go into massive levels of detail as these are documented online (I’ll add links to useful resources at the end of this post), but i’ll discuss my setup and any problems I encounter.

###### Stage 1 - OpenManage Web Server
Setup a windows vm that will contain the OpenManage web server, this doesn’t need to be anything massive. 
![][image-1]
2 vCPU and 2 GB of ram should do the trick.

Download the installer ([here][1]) and run. Installation is very easy, just click threw the windows.

Check it’s succeeded by navigating to *https://localhost:1311*.

Trying to login won’t work as we haven’t installed the OpenManage VIBs (think plugins) on the ESXI hosts yet.

###### Stage 2 - Installing VIBs
I’m going to assume that you don’t have vUM installed (vmware update manager), however if you do just download the VIBs and install them using that then move to **Stage 3**

Otherwise complete the following on each host:

1. Open an SSH session with the host (SSH is disabled by default, but a quick google will reveal how to enable it - do remember to disable it afterwards though).

2. Download the VIB from [here][2] (link to main page incase direct link fails)
	{% highlight bash %}
	wget http://downloads.dell.com/FOLDER02867568M/1/OM-SrvAdmin-Dell-Web-8.1.0-1518.VIB-ESX60i_A00.zip
	unzip
	cd into dir
	install vib
	{% endhighlight %}

[1]:	http://www.dell.com/support/home/us/en/19/Drivers/DriversDetails?driverId=20V28 "here"
[2]:	http://www.dell.com/support/home/us/en/19/Drivers/DriversDetails?driverId=FN2KW "here"

[image-1]:	/static/img/post-images/openmanage/1.png