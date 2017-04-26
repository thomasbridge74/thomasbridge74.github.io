---
layout: post
title: Python issue - installing lxml on AWS EC2 Linux instances
---

While reviewing the one of the videoes as part of [Linux Academy](http://www.linuxacademy.com/)'s
course on python, I tried to install a couple of packages (bs4 (Beautiful Soup) and lxml).   I have two EC2
currently running in AWS - one running Amazon Linux, the other running Ubuntu.   The video suggested that
I should simply do the following:

	pip install bs4
	pip install lxml
	
This didn't work out of the box on either EC2 instance.   

# Amazon Linux

bs4 did install.   I didn't need to do any magic to make this work.   However, when attempting to install lxml I got this:

    compilation terminated.
    Compile failed: command 'gcc' failed with exit status 1
    creating tmp
    cc -I/usr/include/libxml2 -c /tmp/xmlXPathInitUqnUqX.c -o tmp/xmlXPathInitUqnUqX.o
    /tmp/xmlXPathInitUqnUqX.c:1:26: fatal error: libxml/xpath.h: No such file or directory
     #include "libxml/xpath.h"
                              ^
    compilation terminated.
    *********************************************************************************
    Could not find function xmlCheckVersion in library libxml2. Is libxml2 installed?
    *********************************************************************************
    error: command 'gcc' failed with exit status 1
    
    ----------------------------------------
    Command "/usr/bin/python2.7 -c "import setuptools, tokenize;__file__='/tmp/pip-build-DSZymV/lxml/setup.py';exec(compile(getattr(tokenize, 'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /tmp/pip-sqtOYL-record/install-record.txt --single-version-externally-managed --compile" failed with error code 1 in /tmp/pip-build-DSZymV/lxml

It turns out you need two devel packages not installed by default:

		yum install libxml2-devel libxslt-devel

Once that's done, it's installed no problem.

# Ubuntu Linux

## TL;DR version:

Before trying to install the bs4 and lxml pips, type the following:

	apt-get install python-pip libxml2-dev libxslt1-dev
	
## Details

The first issue was that pip didn't work and needed to be installed.  Additionally, a couple of development
packages also needed to be installed.

	apt-get install python-pip
	
At this point it was easy enough to install the bs4 pip.   However, trying to install lxml was more problematic:

	creating tmp
	
	cc -I/usr/include/libxml2 -c /tmp/xmlXPathInitrz9Iy4.c -o tmp/xmlXPathInitrz9Iy4.o
	
	/tmp/xmlXPathInitrz9Iy4.c:1:26: fatal error: libxml/xpath.h: No such file or directory
	
	 #include "libxml/xpath.h"
	
	                          ^
	
	compilation terminated.
	
	*********************************************************************************
	
	Could not find function xmlCheckVersion in library libxml2. Is libxml2 installed?
	
	*********************************************************************************
	
	error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
	
	----------------------------------------
	Cleaning up...
	Command /usr/bin/python -c "import setuptools, tokenize;__file__='/tmp/pip_build_root/lxml/setup.py';exec(compile(getattr(tokenize, 'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /tmp/pip-_sz_5p-record/install-record.txt --single-version-externally-managed --compile failed with error code 1 in /tmp/pip_build_root/lxml
    
Reading the error message suggests installing the libxml2-dev package

	apt-get install libxml2-dev
	
would resolve this only to get the following error:

	src/lxml/includes/etree_defs.h:14:31: fatal error: libxml/xmlversion.h: No such file or directory
	
	 #include "libxml/xmlversion.h"
	
	                               ^
	
	compilation terminated.
	
	Compile failed: command 'x86_64-linux-gnu-gcc' failed with exit status 1
	
	creating tmp
	
	cc -I/usr/include/libxml2 -c /tmp/xmlXPathInitbqq7tG.c -o tmp/xmlXPathInitbqq7tG.o
	
	cc tmp/xmlXPathInitbqq7tG.o -lxml2 -o a.out
	
	error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
	
	----------------------------------------
	Cleaning up...
	Command /usr/bin/python -c "import setuptools, tokenize;__file__='/tmp/pip_build_root/lxml/setup.py';exec(compile(getattr(tokenize, 'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /tmp/pip-x_i1yC-record/install-record.txt --single-version-externally-managed --compile failed with error code 1 in /tmp/pip_build_root/lxml
	Storing debug log for failure in /home/thomas/.pip/pip.log        
	
A quick google established that we also needed to install one more package:

	 apt-get install libxslt1-dev
	 
Once that was done, the module was installed.

# Thought

One of the irritating things in debugging this is that yum / Amazon Linux uses the -devel ending for packages with the
files required to compile the pip whereas Ubuntu uses -dev.   It would be nice if you could havce consistency in naming conventions accross distributions, but clearly even that is too
much to ask!
