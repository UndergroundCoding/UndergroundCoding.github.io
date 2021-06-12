---
layout: post
title:  "Successfully compiling ARMEL."
date:   2021-06-11
ep: 07
categories: radio
---
After a lot of anguish and frustration I was finally able to compile radio targetting armel.
<!--end_preview-->

If you have been following these exciting episodes, full of adventures and rich with drama of the
human spirit you might be wondering "What was the issue, and how did you solve it?"
 
Well, I'm glad you asked. I reserved the next couple of hours just to tell _you_ that.

## Android NDK vs armel-gcc cross compiler
As I may or may not have detailed before, my first approach was the right one: compile radio using
Android NDK's toolchain. The first problem that came up was what seemed like incompatible standard
library. ~~I know this because Tyler knows this.~~ I was getting various errors like this:
 
```
error: undefined reference to 'ost::QueueRTCPManager::dispatchBYE(std::__ndk1::basic_string<char, std::__ndk1::char_traits<char>, std::__ndk1::allocator<char> > const&)'
```
 
However when I ran `nm -D libccrtp.so | grep dispatchBYE` to check whether the symbols existed, or 
if the compiler was trying to deceive me, I saw that they were, in fact there:
```
$ nm -D /usr/lib/arm-linux-gnueabi/libccrtp.so.2.0.6 | grep dispatchBYE
00011f94 W _ZN3ost12RTPQueueBase11dispatchBYEERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
00019b8c T _ZN3ost16QueueRTCPManager11dispatchBYEERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
00019e0c T _ZTv0_n16_N3ost16QueueRTCPManager11dispatchBYEERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
```
 
After several bouts of uncontrollable crying and a lot of time reading I realized what was 
happening: a library missmatch.

Notice that the error says `std::__ndk1::basic_string ...`. Indeed, every reference to the standard 
library was throwing errors. However, looking at the nm output I saw the std references as 
`St7__cxx1112basic_stringIc ...`. The symbol from the compiler error is uses namespace `ndk1` 
whereas the armel gcc cross compiler library is in namesace `cxx11`, or something along those lines.

I assumed the issue was due to Android's NDK having dropped support for libstdc++ (which they refer
as gnustl) since NDK made the full conversion to LLMV compiler tools in version 18, dropping both gcc and
gnustl. I thought I could curb this issue by downgrading my NDK to version 17 where the flag 
`-DANDROID_STL=gnustl_static` specifies clang to use gcc's libstdc++ instead of the default libc++. 

This still gave me the same error but presented slightly differently:
```
error: undefined reference to 'ost::QueueRTCPManager::dispatchBYE(std::string const&)'
```

While it does not have the `ndk1` namespace, it is still not compatible. So I had another brilliant 
bad idea: I will cross compile with arm-linux-gnueabi-gcc and be done with NDK's toolchain. I 
ditched the NDK and tried compiling with 2 flags: `-DCMAKE_C_COMPILER=arm-linux-gnueabi-gcc` and  
`-DCMAKE_CXX_COMPILER=arm-linux-gnueabi-g++`.

And what do you know? It worked! I was flooded with mild euphoria! I had to learn about 
[building order][building-order], but it eventually came together, along with the realization that
my plan wasn't going to work.


My train of thought was along the lines of "I can't exactly make out why Android's gnustl 
(libstdc++) for EABI is not compatible with the arm-linux-gnueabi-gcc library. The back and forth 
between Marek R and SergeyA in this [stackoverflow link][so-what-is-causing-c-function-signature-to-be-different]
did give me some light, though:

Marek R
>on local machine he builds for emulator/simulator and build machine is builds for target platform. This should not have impact on how code works. That is why I ask "why this is a problem?"

SergeyA
>OP explicitly explains why it is a problem for them: I am creating a static library from my C++ code on my development machine and copy it over to the build machine for final linking. OP is doing something which is bound to fail, but you should not be asking them "why is it a problem", since OP explained why such a behavior is an obstacle.

Marek R
>Can you please explain why there is "no guarantee" of ABI compatibility? The ABI is arm. Two NDKs may have different compiler versions. However, ultimately they produce arm-compatible instructions.

I was in agreement with Marek's remarks, but life seems to be in agreement with Sergey's realm, for 
both OP and I are in the same boat.

Finally, the selected answer from Ayxan Haqverdili seems to explain both sides:
>There seems to be two different standard libraries in use, one defines basic_string in an internal namespace called __ndk1, the other one in __1. This is basically an implementation detail. Make sure to use the same compiler and standard library implementation for these to match.

Why different internal namespaces for each implementation? Now that's a war I must fight another day.

## Next steps

I am getting reference errors for every library in use, as they all rely on stl in some capacity. 
This means I need to compile every library in the hierarchy using NDK's toolchain. Lucikly, I
found this very handy explaination [on how to do so with GNU's Autotools autoconfig][ndk-autoconf].

I tried compiling ccrtp with the toolchain but Autotools detected NDK is missing gcrypt. I should 
have known, gcrypt is the lowest dependancy in the chain. I found a possible lead for release bins 
in a GNU mailing list thread titled [Libgcrypt Android Compatibility][libgcrypt-android-release]. If
that doesn't work my next try is compiling libgcrypt with the NDK toolchain.

## Things I learned
* Compiling with shared libraries is easier than with static libraries because there is a dependancy 
order when reading symbols during compilation. However, when _linking_ shared libraries 
the same ordering issue will appear.
* There isn't a way around using toolchains.

[building-order]: https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking/
[so-what-is-causing-c-function-signature-to-be-different]: https://stackoverflow.com/q/66177346/2312929
[ndk-autoconf]: https://developer.android.com/ndk/guides/other_build_systems#autoconf
[libgcrypt-android-release]: https://lists.gnupg.org/pipermail/gcrypt-devel/2018-October/004539.html