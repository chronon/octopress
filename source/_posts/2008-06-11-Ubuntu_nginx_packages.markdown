---
layout: post
title: Unofficial Ubuntu nginx packages
description: "Unofficial Ubuntu nginx packages - no longer maintained"
tags: [dev, sysadmin] 
---

Upate:
------

I am no longer going to be making or updating these packages. Compiling 
nginx from source is very easy, and is still the recommened way to run the 
current version. Clear and simple [instructions are available](http://articles.slicehost.com/nginx) for many Linux distributions. 

Why Did I Make These? 
---------------------

I want to use newer versions than what the Ubuntu universe repository
offers. The feisty package is 0.4.13 (and gutsy will be 0.5.26). Simply
compiling these from source works OK, but involves finding an init script,
setting paths in the configure line, etc. I like the "debianized" features the
official packages offer, so I used the latest debian unstable package (0.5.30)
as a base and made updated packages.

Debian Note: 
------------

I've had a report (thanks Cliff) that my package does not work on Debian/stable due to a dependency 
on a later libc6. An option is to use the Debian source packages backported to Debian/stable
instead. [Details on how to do this](http://www.debian.org/doc/manuals/apt-howto/ch-sourcehandling.en.html) are available...

Changes from the official package: 
----------------------------------

* commented out the first section of the prerm script because it messed 
things up when removing or upgrading.
* changed default port from 80 to 8000 because I use nginx with apache.

Of course use at your own risk. I've done a fair amount of testing and things
work fine for me. I use the amd64.debs on my slice at 
[Slicehost](http://slicehost.com/ "Slicehost"), and the i386.debs at home for testing.

Instructions: 
-------------

**INSTALL:** `sudo dpkg -i nginx_xxxxxx.deb`  
**REMOVE:** `sudo aptitude remove nginx`

Resources: 
----------

* [Official nginx homepage](http://nginx.net/ "nginx homepage")
* [Very useful English nginx wiki](http://wiki.codemongers.com/Main/ "nginx wiki")

Thanks: 
-------

* [Igor Sysoev - nginx developer](http://sysoev.ru/en/)
* Jose Parrella - official Debian package maintainer
* Cliff Wells - [nginx wiki](http://wiki.codemongers.com/Main) maintainer
