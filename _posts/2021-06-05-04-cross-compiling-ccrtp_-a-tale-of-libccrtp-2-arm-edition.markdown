---
layout: post
ep:	04
title:  "Cross compiling ccRTP? A tale of libccrtp 2: ARM edition."
date:   2021-06-05
categories: radio
---
Armed with the beautiful libccrtp I write a simple client/server program to get familiar with both 
the library and the protocol.
I set up wireshark to monitor the transmission of data and after many mistakes I am finally able to 
send a file from the client to the server. Great success! I am now ready to drop this into my 
Android APK and finally get things going!

Except I'm not...

While getting familiarized with Android's native development kit (NDK) I learn that I overlooked a 
pretty big flaw in my portability assessment: I need the c++ libraries built for each [target
triplet](https://wiki.osdev.org/Target_Triplet).

I find that my phone, an old piece of junk, runs on a 32bit ARM processor with hardfloat support. My
brain defaults to "I need to compile libccrtp targetting ARMHF." And that's what I set out to do.

After an obscene amount of time trying to cross compile libccrtp, realizing that for this build to 
work I need all the dependancies also targetting ARMHF, trying to build libucommon from source, I 
finally came across an important finding:

<a href="https://packages.debian.org/buster/libccrtp-dev">
<img src="/assets/img/posts/04-libccrtp-dev_availableArchs.jpg" alt="libccrtp-dev_availableArchs" width="100%">
</a>

Almost like a glitch in the matrix, I am back at the exact same place I was many, many hours ago, 
with only the small intellectual advance that I need to target ARM architecture. Still, progress is
progress. Let's download this:
```
$ sudo apt install libccrtp-dev:armhf
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package libccrtp-dev:armhf
```

WHAT? Ok, I need to add the architecture to `dpkg`s arch list. Fine, let's do it:
```
$dpkg --add-architechture armhf
```

Awesome, now let's get that library!
```
$ sudo apt install libccrtp-dev:armhf
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package libccrtp-dev:armhf
```

Hmm. Still nothing? Well, let me save you the headaches and frustrations...

After adding a new architecture to `dpkg` we need to update `apt`'s package list on and then apply 
the updates. To do this we just need to call:
```
apt update && apt upgrade
```

Now apt should have pulled the list of armhf packages, so we try, again:
```
$ sudo apt install libccrtp-dev:armhf
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
...
```

Yes! It worked! We can verify by checking the contents of the package:
```
$ dpkg -L libccrtp-dev:armhf | grep libccrtp.so
/usr/lib/arm-linux-gnueabihf/libccrtp.so
```

Ok, finally! I am ready! Watchout Android, here I come!
