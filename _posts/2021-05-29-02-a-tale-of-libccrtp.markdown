---
layout: post
ep:	02
title:  "A Tale of libccrtp."
date:   2021-05-29
categories: radio
---
The project started. I set out to work with what seems like the only library for RTP and RTSP 
protocol. GNU's ccRTP library is exactly what I need, but in C++ instead of Java. However, it does
have a benefit over any Java product: it will be much lighter, and portable. Since it is my 
intention to have Radio run on various types of devices, even very small one, and to have various 
servers on a cloud, a lightweight program will help keep costs low and provide easy scability. In 
this fashion I decided to build a C++, Linux program.

I decide OK, ccrtp it is. I hit Download from GNU's website and what I downloaded was the source 
code. It did not dawn on me that the source was not the binaries, and I simply assumed I had to 
compile it.

Long story short, I have to go through hours, days, of learning how to compile from source on Linux.
I learn about `configure`, I _start_ to learn about `cmake`, and I spend  a very long time trying to
find the dependencies for compiling libccrtp. It took me very long to find and download 
libcommoncpp, libucommon, gcrypt, etc. I finally get a handle of `apt`, learn the purpose of 
`configure`, and get `Makefile` to compile successfully.

Then, late at night in bed, feeling proud of myself and finally ready to start actually coding, I 
come across the thought that "maybe, just maybe, I should search for the ccrtp's _binary_." What I
found was both funny and disappointing:
``` 
$ apt search libccrtp
Sorting... Done
Full Text Search... Done
libccrtp-dev/stable 2.0.9-2.3 amd64
  Common C++ class framework for RTP packets - development files

libccrtp-doc/stable,stable 2.0.9-2.3 all
  Documentation files for  GNU ccRTPp library

libccrtp2v5/stable,now 2.0.9-2.3 amd64 [installed,auto-removable]
  Common C++ class framework for RTP packets
```

Whatever. At least I learned a lot along the way.
