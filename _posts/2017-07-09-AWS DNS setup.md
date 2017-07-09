---
layout: post
title: DNS Server for AWS Advanced Network Specialist learning
---

Yesterday I passed the AWS Advanced Networking Exam.   I'll post some thoughts on the exam in a couple of days, but
in the meantime I thought I'd share what I learned from experimenting during the learning process around DNS setups.   I 
used the [Linux Academy](http://www.linuxacademy.com/) materials, and as part of that course they talked briefly around
DNS setups.

I decided therefore to do a simple setup to experiment with this.   The requirement is that:
1. There is an internal DNS zone for internal use only (called awscloud.local).   This domain is managed from within Route 53.
2. All clients need to resolve both external and internal domains.
3. Not all clients will be in the local VPC - some will be in a peered VPC or could in theory be in the onpremises network needing
to resolve internal domains.

The high level solution is simple: Spin up an EC2 instance (actually, more than one for resiliency), run a caching DNS service
on it, and then forward queries for the local zone to the DNS server for the VPC.

For those of you who have forgotten, or weren't aware, there is a DNS server available by default on the second IP address of
the VPC (note: the VPC - not the subnet).    For this example, my VPC in London is using the subnet 10.44.0.0/16 so the DNS
server is on 10.44.0.2 - regardless of the subnet within the VPC your client instance is in.   However, for the basis of this 
note I assumed all clients would be in 10.0.0.0/8 - not just the local VPC, but other networks with access to this VPC.

The simple way to set this up was to spin up an instance of the Amazon Linux AMI.    Once this is spun up, the first thing to 
do is install the packages:

	sudo yum install -y bind-utils bind
	sudo service named start
	sudo chkconfig named on
	
The next thing to do is to change the configuration of the DNS server.   To do this, edit /etc/named.conf and update the
main options as follows:

	listen-on { any; };
	allow-query     { 10.0.0.0/8; };
	dnssec-validation no;           
	
This is because by default the name server under Amazon Linux only listens on the loopback address, doesn't allow caching queries
from anywhere else and insists on validating DNSSEC for all entries.   The first two make sense for security reasons - caching
servers need to be restricted because of the potential for DNS amplification attacks) but the last one frankly is insane as it
results in queries forwarded to Route53 not being returned 
[because Amazon don't support DNSSEC for query validation](https://aws.amazon.com/route53/faqs/#support_dnssec).   
Finally, setup forwarding for the local zone to Route53 by appending the below to the file.

	zone "awscloud.local" {
		type forward;
		forward only;
		forwarders {
			10.44.0.2;
		};
	};
	  
And then restart the server:

	sudo service named restart
	
Once this is done, any client that can route to the EC2 instance can resolve both the external and internal domains by
using the EC2 instance for their DNS resolver.	  
