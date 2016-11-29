---
layout: post
title:  How to install PiHole on Ubuntu 16.10 with Active Directory support
date:   2016-11-29 21:45
categories: [vmware, homelab, pihole, network]
---
#### What is PiHole?
So from the website [https://pi-hole.net]() they claim that it’s ‘Network-wide, hardware ad blocking’. In reality it’s a wrapper around a linux dns server (dnsmasq) that uses black lists to send requests for *known bad or advert* websites to a *black hole* or to you and me *0.0.0.0*, this means that the ad isn’t rendered, so shouldn’t be displayed within the browser / mobile app.

The PiHole package includes a web interface that gives you daily stats (amount of ads blocked, percent ads etc) but also *white listing*, this is especially important for on demand tv websites that may check their ads are rendered. That’s one downfall to the PiHole system, it can’t avoid ad block detection systems, so a good comprise would be to pair it with a browser based solution where possible.
  
Originally PiHole (as the name implies) was designed to be ran on a Raspberry Pi, while that’s great and all, I’ve got a vSphere cluster that needs using, so I’ll be deploying it onto a *Ubuntu 16.10* with a fairly low resource allocation. The script should however work for any Debian / CentOS based distro.

#### Installing:
Open an SSH session to your vm / pi / smart toaster / whatever and run the following command:
{% highlight bash %}
curl -L https://install.pi-hole.net | bash
{% endhighlight %}
> make sure it’s ran as root! (Prefix the command with sudo) 
After a little while PiHole will be setup and you can login to the web interface to make sure it’s all okay.

The web interface is accessed at the following:
*http://xxx.xxx.xxx.xxx/admin/index.php* or http://pi.hole/admin (assuming you’ve skipped ahead and set your network to use the new dns)

#### Setup
I’m running my AD on Windows Server 2016, this provides my internal DNS for my domain clients. The idea is that we can add the new PiHole dns to become the *forward lookup* for the AD’s dns, replacing Google’s *8.8.8.8* as the primary forward lookup. I will however leave it as the secondary, just incase the PiHole ever goes down.

This is actually very very easy to do with the following steps:
![][image-1]
Execute the install command (making sure to use *sudo*) in either an SSH session or at the local machine.

![][image-2]
After a short time the installer will display a graphical window with mostly click threw screens, but it is worth checking the info it provides.

![][image-3]
Make sure to set a static ip for the server (I’ll post a quick guide on that soon)

![][image-4]
Double check that the correct network interface is selected, mine is a virtual interface hence the ens160 naming.

![][image-5]
I selected just ipv4, but if you’ve got a v6 capable network / isp, then feel free to enable that too.

![][image-6]
Double check the network info 

![][image-7]
I selected Google for the upstream dns, but you can select any or provide your own.

![][image-8]
Take a note of the ip and the method of reaching the web Interface (it is however always available at *http://pi.hole/admin* from any machine served by the PiHole’s dns.

![][image-9]
The Installer will drop back to the terminal and reiterate the successful installation.

#### Active Directory Integration:
Open up your DNS server manager (Administrative tools \> DNS)
![][image-10]
Right click on your server’s name and select properties.

![][image-11]
Go to the ‘Forwarders’ Tab

![][image-12]
Add the ip of the PiHole appliance as the top most DNS, with Google (or whoever) below, this will make sure that your client’s still have working DNS resolution even when the PiHole goes down.

![][image-13]
Go to any client machine (served by your AD) and enter *http://pi.hole/admin*, you’ll be presented with the web interface.



[image-1]:	/static/img/post-images/pihole/1.png
[image-2]:	/static/img/post-images/pihole/2.png
[image-3]:	/static/img/post-images/pihole/3.png
[image-4]:	/static/img/post-images/pihole/4.png
[image-5]:	/static/img/post-images/pihole/5.png
[image-6]:	/static/img/post-images/pihole/6.png
[image-7]:	/static/img/post-images/pihole/7.png
[image-8]:	/static/img/post-images/pihole/8.png
[image-9]:	/static/img/post-images/pihole/9.png
[image-10]:	/static/img/post-images/pihole/10.png
[image-11]:	/static/img/post-images/pihole/11.png
[image-12]:	/static/img/post-images/pihole/12.png
[image-13]:	/static/img/post-images/pihole/13.png