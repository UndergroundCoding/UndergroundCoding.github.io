---
layout: page
onHeader: false
title: Radio
permalink: /todos/radio.html
---

# TODO
- [ ] Check if Queuable is necessary. If not used by anyone else, probably thrash
- [ ] Write tests
- [ ] Seperate Blog and Radio todos (currently using this file for both)

# DONE/NA
- [ ] Switch all instances of pointers to smart pointers
	 * No pointers at this time.
- [ ] Move all classes to abstract/concrete
	* Too complicated at this point and not necessary.
- [x] Make EventHandler RAII
- [x] Implement condition_variable instead of busy loop
- [x] Namespace EventHandler. The typedefs are global and polluting
	* Moved typedefs to class scope, but had to leave some global. Must revisit later
- [x] Createa an async_queue class to wrap queue.
- [x] At some point we will need to put some protection to avoid 2 threads waitingOnEmpty to run at
 the same time. Otherwise 2 consumers might run into a data race. This can be done by using 
 notify_one(). 
	* Ended up creating a WaitType::Types and 2 condVars to resolve it.
- [ ] Create async_vector to wrap std::vector and implement in Scheduler
	* Not needed. The only vector in used is on scheduler. Each thread tracks one event which is 
	already thread-safe and none of them share resources. If they do, they manage it themselves.
- [ ] Get rid of scheduler eventMap and let the caller provide the scheduling function.
	* The entire scheduler has been replaced for async calls
- [x] Replace scheduler with std::async()
- [x] Migrate/link TODO.txt with UndergroundBlog as a page
