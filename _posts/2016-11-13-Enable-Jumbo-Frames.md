---
layout: post
title:  Jumbo Frames
date:   2016-11-11 00:45
categories: [vmware, homelab, networking]
---

#### Jumbo Frames?
Wikipedia defines a *jumbo frame* as an Ethernet frame with more than *1500 bytes of payload*, the limit set by the IEEE 802.3 standard. A jumbo frame can carry up to 9000 bytes of payload.

#### Things to note
You need to make sure that **every** device in the network / vlan supports jumbo frames and is configured with the same MTU.
 
It shouldn’t have too much of a negative effect if you had a device that had an MTU of say 1500, with other devices being at 9000, but for the sake of sanity it’s recommended that they’re all the same.

#### Setup
So I’m setting up jumbo frames on my 1gb ‘vmStorage’ vlan (until I setup a proper storage device and 10gb network), this vlan will only be used for storage traffic - currently NFS3.

My networks core currently comprises of Cisco SG-300-x series switches, these have been ideal and have loads of features, but I will probably replace them as I switch to 10gb.

I’ve got three of these switches, two 10 port PoE models and one 28 port (this is the primary switch for the server rack). Each switch is identically configured, the jumbo frames setting is found under ‘Port Management’ (funnily enough).
![][image-1]

> You do have to enable jumbo frames globally - you can’t do it per vlan (annoyingly). But after doing a fair amount of homework it looks like this is perfectly safe, it simply raises the upper limit to 9000.. not enforcing everything to 9000.

Be sure to repeat the process for any switches you have on the direct path (I did not need to configure other access switches as the vlan is not accessed outside of the core).

The vSphere networking environment also needs to be configured to be allowed to use the max MTU of 9000, this is however very simple.
![][image-2]

For the purposes of this test I haven’t changed any of the virtual adapters to the higher MTU, but this is definitely something you’ll want to do if you want your vmkernal to run over the increased bandwidth. I’m just going to spin up a few test vms for the time being.

#### Basic Testing
So to make sure I’d setup jumbo frames properly I span up some Ubuntu 16.04 virtual machines, a total of 4 in fact, spread across two hosts.

Node1:
- JumboTest1 (vmStorage)
- NormalTest1 (testNet)
Node2”
- JumboTest2 (vmStorage)
- NormalTest2 (testNet)
The jumbo test vms were on the ‘vmStorage’ vlan (not used for anything else) and the normal test vms were on the ‘testNet’ vlan, again not being used.

On each of the jumbo test vms I configured a static ip as follows:
> Be sure to change the ‘address’ on each vm, otherwise you’ll end up with some nasty conflicts.
*(/etc/networking/interfaces)*
{% highlight bash %}
iface ens160 inet static
address 10.55.0.10
network 10.55.0.0
gateway 10.55.0.1
net mask 255.255.255.0
mtu 9000
{% endhighlight %}
Notice the added *mtu 9000* on the last line, restart networking (or reboot) to enable the new configuration.

Once both JumboTest1 & JumboTest2 are configured with static addresses and a 9K MTU you can do quick ping to confirm it’s all working.
{% highlight bash %}
ping -M do -s 8972 \<JumboTest2\>
{% endhighlight %}
> On Linux the ping / ICMP implementation doesn’t encapsulate the 28 byte ICMP header, therefore we mist take the 9000 and subtract 28 = 8972.

The result should just look like a normal ping, however if you get something like this:
{% highlight bash %}
PING xx.xx.xx.xx (xx.xx.xx.xx): 8184 data bytes
ping: sendto: Message too long
{% endhighlight %}
Then you’ve not enabled jumbo frames on your client.

If you see this:
{% highlight bash %}
PING xx.xx.xx.xx (xx.xx.xx.xx): 8184 data bytes
Request timeout for icmp_seq 0
{% endhighlight %}
You’ve enabled jumbo frames on your client but not your target (or a switch on the way).

#### Speed tests
Now that we know jumbo frames are configured correctly we can run some speed tests  using the **iperf3** tool.

On all of the vms run the following commands:
{% highlight bash %}
sudo apt update
sudo apt install iperf3
{% endhighlight %}

Open an SSH session with jumboTest2 and run the following command:
{% highlight bash %}
iperf3 -s
{% endhighlight %}
This creates a listener ready for the speed test.

Similarly open another SSH session, but this time to jumboTest1..
{% highlight bash %}
iperf3 -c \<jumboTest1-IP\>
{% endhighlight %}

After a couple of seconds you should see some results:
![][image-3]

Repeat the commands on the respective ‘normal’ test vms, you should get something like this:
![][image-4]

So that’s around a 70Mbits/sec difference with exactly the same hardware and little work. In part two I’ll conduct a test using a disk benchmark tool and network storage.

##### To be continued…

[image-1]:	/static/img/post-images/jumboframes/switch-jumbo.PNG
[image-2]:	/static/img/post-images/jumboframes/9000mtu-vsphere.PNG
[image-3]:	/static/img/post-images/jumboframes/jumbo.png
[image-4]:	/static/img/post-images/jumboframes/normal.png