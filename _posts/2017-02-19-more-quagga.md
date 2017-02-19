---
layout: post
title: More Quagga Notes
---

Following on from [yesterday's blog](/2017/02/18/Ubuntu-GNS3-tutorial.html) about Quagga setups here are some more 
thoughts that might be of interest.

# User accounts

I hinted at it in yesterday's report, but you do not need to be root to make configuration changes to Quagga.   In fact,
all you need to do is to be a user who is in the quaggavty group.     If you have a non user account there are a couple of ways
to add a user to the group.    One way is to directly edit the /etc/group file, but as we already setup a Ubuntu environment
it's safer to add the user to the group as follows:

	sudo usermod -G quaggavty <username>
	
Alternatively, you might want to set up a user account that can only access the Quagga environment when logged (making the
experience even more Cisco like!).     As an example, I've set up the account "neteng" below to do this:

	sudo useradd -G quaggavty neteng
	sudo passwd neteng    
	sudo chsh -s /usr/bin/vtysh neteng
	
One issue with this is that in the Ubuntu environment the effect is to use "less" as the pager of choice, requiring
regular hits of the q key when interacting with Quagga (which given the purpose of this excercise is partly to make
things more Cisco like seems sub optimal).    The fix here is to add the line

	VTYSH_PAGER=more

to the /etc/environment file.     Then when user "neteng" logs in, they'll be taken direct to the vtysh environment and
not the bash shell.   You can still escape out to a shell using the command

	start-shell
	
(which while I've not seen this in the Cisco world, is very similar to the behaviour of the Citrix Netscalar where although
you're logged into their management shell, it's easy to escape into a proper UNIX like environment).

# Configuration files

The full running configuration from the lab is here:

	!  
	interface enp0s8
	 description ethernet1
	 ip address 192.168.3.130/30
	 ipv6 nd suppress-ra
	 no link-detect
	!  
	interface enp0s9
	 description ethernet2
	 ip address 192.168.3.134/30
	 ipv6 nd suppress-ra
	 no link-detect
	!  
	interface enp0s10
	 description ethernet3 - BGP Peering
	 ip address 172.16.1.1/24
	 ipv6 nd suppress-ra
	 no link-detect
	!  
	interface lo
	 no link-detect
	!  
	router bgp 65001
	 bgp router-id 192.168.3.134
	 network 192.168.0.0/22
	 neighbor 172.16.1.3 remote-as 65002
	!  
	router ospf
	 redistribute bgp
	 network 192.168.0.0/22 area 0.0.0.0
	!  
	ip route 192.168.0.0/22 Null0
	!  
	ip forwarding
	!  
	line vty
	!  
	end          
	
This is actually split over three files:

	/etc/quagga/bgpd.conf          
	/etc/quagga/zebra.conf   
	/etc/quagga/ospfd.conf
	
## Zebra Configuration

The contents of zebra.conf are:

	!
	interface enp0s8
	 description ethernet1
	 no link-detect
	 ip address 192.168.3.130/30
	 ipv6 nd suppress-ra
	!
	interface enp0s9
	 description ethernet2
	 no link-detect
	 ip address 192.168.3.134/30
	 ipv6 nd suppress-ra
	!
	interface enp0s10
	 description ethernet3 - BGP Peering
	 no link-detect
	 ip address 172.16.1.1/24
	 ipv6 nd suppress-ra
	!
	interface lo
	 no link-detect
	!
	ip route 192.168.0.0/22 Null0
	!
	ip forwarding
	!
	!
	line vty
	!
 
This shows that the zebra daemon is the one that sets the IP addresses on the interfaces and is also where static routes are
configured (incidentally, enp0s3 was also used in this lab environment to allow me to ssh into the Quagga VM as cut and paste
doesn't work from the VirtualBox console - that was configured directly in Ubuntu's /etc/networks/interfaces file.   This 
IP configuration therefore does not show in the zebra.conf file.   We can check that the IPs configured also are configured directly on the interfaces:

	thomas@ubuntu:~$ ifconfig enp0s8
	enp0s8    Link encap:Ethernet  HWaddr 08:00:27:16:35:31  
	          inet addr:192.168.3.130  Bcast:192.168.3.131  Mask:255.255.255.252
	          UP BROADCAST MULTICAST  MTU:1500  Metric:1
	          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000 
	          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
	          
There are server scenarios in which it would can be useful to define static routes other than the default gateway.   The above
would suggest just running Quagga's zebra daemon and adding interface and static routes via vtysh might be one way to manage
this.

Also in the configuration above is line vty - this is obviously something worth looking into in due course (so watch this space!).    It
is not inherently obvious why Quagga has line vty configuration in the same style as Cisco - the vtys in Cisco are what terminate
remote ssh sessions (or even telnet on really, really old Cisco routers).    Typically for Quagga that is handled by the underlying
Linux OS, so it raises the question of what you would do within the Quagga configuration to manage vty access.

## OSPF Configuration

The contents of ospfd.conf is:

	!
	interface enp0s8
	 description ethernet1
	!
	interface enp0s9
	 description ethernet2
	!
	interface enp0s10
	 description ethernet3 - BGP Peering
	!
	interface lo
	!
	router ospf
	 redistribute bgp
	 network 192.168.0.0/22 area 0.0.0.0
	!
	line vty
	!

Clearly the OSPF configuration is contained within the file.    Interestingly though we see the interfaces and their
descriptions effectively repeated from the zebra.conf file.    There is no obvious logic for this - enp0s10 isn't running 
OSPF.    We can also see the line vty section is repeated.  

The main thing to note about the OSPF configuration is that here we use the network &lt;CIDR&gt; area &lt;area&gt; - whereas in a 
Cisco environment we would use network &lt;ip&gt; &lt;wildcard&gt; area &lt;area&gt;

## BGP Configuration
 
The contents of bgpd.conf are:

	!
	router bgp 65001
	 bgp router-id 192.168.3.134
	 network 192.168.0.0/22
	 neighbor 172.16.1.3 remote-as 65002
	!
	line vty
	! 
	
A couple of things to note here:
* Quagga has clearly grabbed an interface IP to act as the router ID - so in a
production environment one might want to excercise more control over the configuration.     
* The network statement
works in CIDR form rather than Cisco's network &lt;network&gt; mask &lt;netmask&gt; style.   (But this isn't that surprising when looking at
the OSPF configuration).

