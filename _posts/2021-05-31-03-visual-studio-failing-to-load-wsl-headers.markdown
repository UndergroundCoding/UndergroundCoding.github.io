---
layout: post
ep: 03
title:  "Visual Studio failing to load WSL headers."
date:   2021-05-31
categories: radio
---
Another day, another bump on the way. This bump, however, was more of a warning than an error and,
while I was able to proceed without fixing it, the consequence was very annoying.

When I setup my Visual Studio cmake project, specified it will be cross-compiling using Windows 
Subsystem for Linux (WSL), and specified the include directories on CMakeLists.txt, Visual Studio
seemed to accept everything fine and well (no errors or warnings), and I was able to compile my 
project without issues, however Visual Studio just couldn't seem to find the proper header symbols.
Everything had a red squigly line under it.

After much research I finally began to understand what Visual Studio does to load the symbols, and
why it was able to compile but not resolve any symbols locally.

When creating a cmake project, in the `CMakeSettings.json` file there is an optional entry:
`remoteCopyAdditionalIncludeDirectories`. Visual Studio goes into the directories specified in this
entry and copies all files to a local directory, and uses those files to resolve any necessary 
symbols. The files are stored to `%USERPROFILE%\AppData\Local\Microsoft\Linux\HeaderCache\1.0`.

When I navigated to that directory, there was a single directory `wsl_debian`, which is the system 
I am using for the compiling the project. Inside the directory, however, there were none of the 
expected header files. For some reason, Visual Studio simply refuses to copy the necessary files 
from the WSL system to this directory, so I tried a different approach: using Visual Studio's remote
 system option.
 
Visual Studio's remote system allows the user to develop on one machine and compile on another, 
remotely. It is essentially what it is doing with the WSL setup, but using SSH to communicate with
the remote server. The logic is the same: copy the remote includes to a local directory so 
intellisense can do its intellisensing.

I installed the SSH daemon on my Linux (WSL) system and set up the remote system connection on 
Visual Studio. I then change the CMakeSettings to use the remote system instead of WSL and let 
Visual Studio and CMake do their thing.

I checked the header cache directory again and found that this time there were 2 folders: the WSL 
one that already existed and another one with random numbers. The random number one, at last, had 
the header files I needed. I copied the header files to the WSL directories and everything worked,
finally.

So, the moral of the story here is: just copy the necessary headers into the local header cache 
directory. There is no need for the remote system setup, that's just how I managed to get to this 
conclusion.
