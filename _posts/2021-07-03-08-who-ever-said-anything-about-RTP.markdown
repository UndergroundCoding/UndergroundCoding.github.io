---
layout: post
ep: 00
title:  "Who ever said anything about RTP? What I actually meant was..."
date:   2021-07-03
categories: radio
---

After I was finally able to compile the code on a linux host targetting ARMEL, I decide the there 
really is no point in going forward with the RTP route this early in the game. I am going to need
a TCP pipe regardless, so I decided I would save myself a lot of trouble by starting with it.

I once read someone saying that before choosing to use UDP one should first implement and test with
TCP as for most situations it is easy to implement and gets the job done just fine.

Well, I decided to listen and take the path of least resistance. I am finally able to start writting
code and writing code is what I have been doing.

<!--end_preview-->

# The Plan
I have drawn out the following strategy with an immense touch of artistic skill:

<img src="/assets/img/posts/08-thePlan.jpg" alt="thePlan" width="100%">

## The Scheduler
I will start out by describing the central system of the project. The scheduler is, like the name
suggests, the component in charge of deciding what the system does next.

Unlike procedural systems, where once instruction goes after the other, Radio is event-driven, 
meaning that different components act on demand through some input. In this case the inputs will be
the user pressing a button, such as record, or the server alerting the system to do something, or
the audio player starting the next song. There is no correct order of operation, things need to run
concurrently. This is where the scheduler comes in. Its duty is to decide when an event's action 
gets triggered, and actually run it.

Let me try to clarify what the scheduler does with an example.

Suppose the system wants to request 5 songs from the server to keep the audio-queue populated. The
system must trigger a `requestAudioFromServer(5)` action. But lets also assume the system wants to
upload a cool audio Jimmy just recorded. Instead of waiting for the request above to finish, which 
can take long, it just schedules the event on the scheduler and goes on to worry about other things.
In this case, "other things" means uploading Jimmy's audio (or, rather, scheduling the even to do 
so).

It might seem unnecessay to have this scheduler since the system can just open a thread to handle
each action/request, but then each module would need to do the annoying task of managing a thread 
pool. The scheduler takes control of every thread creation/destruction and allows granular flow 
control in a threaded environment.

The real magic of the scheduler is that it allows the server to act as a proxy, scheduling events
that belong to other modules.

### AsyncQueue

Radio runs on a qeue. Audio is pushed and popped from a queue, and nearly every component either 
fetches data from the queue (player) or pushes data into the queue (recorder, server). Since each 
component is threaded, it is clear that we need a thread-safe queue. Unfortunatelly the standard 
library (STL) does not offer one. I had to write a thread-safe wrapper for the STL queue.

#### Understanding Thread Safety