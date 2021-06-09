---
layout: post
ep:	05
title:  "NDK, toolchains, and a lot of headaches. A tale of libccrtp 3: ARM*EL* edition."
date:   2021-06-05
categories: radio
---
_A new day new day. A day for a new battlefield; for new conquests.\"_

I began that day with positive thoughts, for I positively thought that I had conquered the war of 
the compilation. Very early into this beautiful, hopeful day for, however, I realized that the 
battle I had won was just another one in this war that seemed never to end.

The idea was simple, I thought. _\"Tomorrow I will compile libradio.dll for amrhf, drop into the 
Android project, and start the real work of developing!\"_ I slept like a baby and looked forward 
for the new day.

I compile my lib, spend a large amount of time finding out how to drop it into Androi project, drop 
it into Android, hit compile, and wait.

...

Doesn't work. The gradle starts to complain about my library. Cannot find sources, doesn't recognize
 functions, typical compiler BS.
 
Everything points me towards using Android NDK (native development kit). Sigh... Ok, another step, 
another lesson, and another set of frustrations at the compiler battle field.

I am back at cmake; reading, researching, learning about toolchains, ABIs, and other various things.
I find that things are potentially easy (they were not), and that I might finally be at the edge of 
victory again (I was not).

I try to compile and I am faced with an error similar to this:

```
error: xxx uses VFP register arguments, xxx does not
```

What? I reasearch more and, basically, Android NDK uses armeabi and does not support hard floating 
point. So what does this mean? It means I have the wrong libs. What I have, referred by `apt` as 
`armhf`, is `arm-linux-gnueabihf` and what I need is `arm-linux-gnueabi`, which `apt` refers to as 
`armel`.

Basically the rest of the story here is Episode04 with every reference to `armhf` replace with 
`armel` and a lot less time researching.
