---
layout: post
title: DNS Introduction for www.linux.ie
---

Some years ago (many, many years ago it would appear...) I wrote an article on DNS for the website
of the [Irish Linux Users Group](http://www.linux.ie).   The site itself appears to have been offline since 2013
and I've no idea if they still have the original article.   Luckily however the 
[Internet Archive](http://web.archive.org/web/20000424001955/http://www.linux.ie/Articles/dns.html) still has a copy,
which I've pulled, updated to Markdown and now include here.   

I feel the article has aged pretty well - although some of the links at the end are now grossly out of date, and the
O'Reilly book referenced below had its [fifth edition](http://shop.oreilly.com/product/9780596100575.do) published in 2006
the way resolvers work hasn't changed in the time since I wrote this article.

* TOC
{:toc}

# Structure of the Domain Space.
The domain namespace is organised in a similar structure to the UNIX filename space. You start at the top of the tree at the 
root. Immediately below the root at the Top Level Domains. These consist of country specific ones and seven generic Top Level 
Domains or gTLDs.
Below these domains your have the second level domain names. These domain names are usually "delegated" by the administrators 
of the relevant TLD which means that someone else is responsible for administering that part of the name space (eg the 
administrators of .ie delegated the domain linux.ie to the Irish Linux Users Group, which means that ILUG are now responsible 
for administering the domain in any way they see fit. without reference to the administrators of .ie).
Once a domain is delegated, the administrators of the domain are responsible for making changes within that domain.

# Domain Name Resolution:
Domain Name resolution works on the model of resolver and server. A resolver sends a query to a server which returns
the answer to the resolver. 
If the server already knows the answer (either because it has cached the answer or because it is the authorative server for 
that domain), it will return that answer to the resolver, otherwise it will query other servers until it finds the answer.
The algorithm for searching is quite straight forward. The server starts by querying the root servers, which will respond with 
one of three possible answers:

* The answer to the query
* A not known answer.
* A referral to a list of name servers which have more information on the domain.

The reaction of the server making the query in each of these is:
 
 * Return the answer to the resolver.
 * Return the not known answer to the resolver.
 * Send the query to the list of name servers and repeat until we get an answer or a not known answer.
 
One important thing to note about this process is that any information the server receives is cached for a period. This period 
is known as the TTL, and is usually set by the domain administrator.
Any further requests for information during the TTL is then answered from the DNS cache.

Example:
Imagine we have a newly started server, which has no cache. This server is authorative for one zone, linux.ie. For our example 
we will use four lookups - one for each of the following domains:

* www.linux.ie
* www.via-net-works.ie
* www.tcd.ie
* www.microsoft.com

Firstly our resolver asks for www.linux.ie. Our server is authorative for this domain, so it answers from its own set of 
information.
Secondly we ask for www.via-net-works.ie. Our server knows nothing about the domain, so it sends a request off to a root 
server. The root server will send an answer back saying that it doesn't know about www.via-net-works.ie and providing a list of 
servers that are authorative for the .ie domain. The server then sends a request off to one of these servers, which in turn 
will answer that it only knows the list of servers for via-net-works.ie. The server finally sends the request to one of thse 
servers which then anwer the question.

Next the resolver asks the server for www.tcd.ie. As the server already knows the .ie domain servers (it picked this up last 
time) it sends a request off to these servers asking for www.tcd.ie - the servers will then respond with a list of servers for 
tcd.ie, which will in turn be queried and hopefully will return an answer.

Finally we ask for www.microsoft.com - our server knows nothing about this, so asks the rootservers which will respond with a 
list of .com nameservers. This will be queried and will return a list of microsoft.com servers which will be queried for the 
final answer.

# Configuration of a resolver under linux.

For ordinary DNS resolution (eg for a dial up machine), you need to configure your resolver. Before you do this, you should 
know the list of DNS servers for your ISP. For this example I will use the numbers 192.168.100.1 and 192.168.100.2
Note that these numbers will not work with any ISP, you must ask them for the right DNS servers.

To configure the resolver, you need to edit the file /etc/resolv.conf. There are a number of options you can enter into this file, here I will describe the more common ones.

	nameserver

You can have up to three name entries in the file. Each nameserver command must be followed by the IP address of a nameserver. 
Nameservers are queried in the order that they are listed. If there are no nameserver entries, the server on the local machine 
is used.

	search 

Search list for domains. This is a list of domains that will be automatically added to the end of a hostname. For example, if 
you have the directive search maths.tcd.ie then the command "telnet salmon" will take you to salmon.maths.tcd.ie.
You can have up to six entries in this list. Note that this can generate a lot of network traffic so it is probaby not a good 
idea to do this for domains that don't have servers on the local network.

The resolver uses the following method to determine whether or not (and when) to use the searchlist.

Does the domain name contain > n dots?
Try to resolve this as an absolute name.
If it doesn't contain the specified amount of dots, or resolve to an absolute name, try the search list.
The default value for n in step 1 is one (ie any name containing at least one dot will be treated as an absolute name first). 
You can change this behaviour by adding the following line to the conf file:

	options ndots:n 

where n is the minimum number of dots that will be treated as an absolute name first.

Further reading:
1. DNS and BIND, 3rd, O'Reilly Books
2. [DNS Resources Directory: http://www.dns.net/dnsrd](http://www.dns.net/dnsrd)
3. resolver(5)
4. [BIND Home Page; http://isc.org/bind](http://isc.org/bind)
5. DNS HOWTO

