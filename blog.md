# The Underground Blog
## Episode 1: I Start a blog
Today I started a blog. I'm not sure if it will be program specific, or dev specific, or specific at
all. I don't even know if I will keep itreading, but I will at least write and commit this entry.

This blog is (will be) hosted on github, initially as a markdown file but later as a website 
(through github pages).

Anyway I've been wanting to list my struggles; my learnings;-and-; my victories. It is important to
anchor your path by remembering the beginnings.

## Episode 2: A Tale of libccrtp.
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

## Episode 3: Visual Studio failing to load WSL headers.
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

## Episode 4: Cross compiling ccRTP? A tale of libccrtp 2: ARM edition.
## Episode 5: Can we please talk about Android development?
## Episode 6: Avoidance; or; How the blog came to be.