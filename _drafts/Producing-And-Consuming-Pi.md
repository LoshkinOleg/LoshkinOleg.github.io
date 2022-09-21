---
title: "Multithreading in C++: Producing and Consuming PI"
date: 2022-09-10
layout: post
categories: blogpost
permalink: /:categories/:title
---

Hi, I’m Oleg, a Junior Game Audio Programmer and this is a continuation to [my previous blogpost on multithreading](). In the last post, I've shown you how to kick off threads and asynchronous tasks without getting into the details of memory sharing. Well, now's the time to get aquaited with the spooky side of multithreading!

In this blogpost, I'll introduce you to the concepts and challenges of resource sharing in a multi-threaded program and show you the basic C++11 facilities that are commonly used for writing portable multithreaded programs.

By the end of this blogpost, you'll hopefuly understand the bare minimum required to write multithreaded applications and will have a good list of learning references you can explore to learn more.

Note that in this blogpost, I will not focus on any hardware details relating to multithreading, but I encourage you to read on the subject since all of the constraints present when solving multithreading challenges come from there. Once again, I recommend [Jason Gregory's "Game Engine Architecture"]() for this.

-----

<h1>Credits</h1>

Before going any further, let me credit the people, software and learning resources I've used in the making of this blogpost:

The [practical project / exersice]() I've put at your disposition could not have been possible without the following, in no particular order:
- [Visual Studio 2022 (windows)]() and [VSCode (linux)]()
- [Git]() and [Github]()
- [CMake]()
- Sergey Yagovstev’s [easy_profiler]()
- [Fabrice Bellard's implementation](https://bellard.org/pi/pi1.c) of the [Bailey-Borwein-Plouffe formula]() for generating digits of PI

Although not all explicitly mentioned in the content of this blogpost, the following learning resources were very useful in the making of this blogpost and I encourage you to read / watch them yourself:

- [www.modernescpp.com's explanation of Condition Variables](https://www.modernescpp.com/index.php/c-core-guidelines-be-aware-of-the-traps-of-condition-variables)
- My personal bible: [“Game Engine Architecture” by Jason Gregory]. Most of the information presented in this blogpost can be found in more extensive details in that book alongside a slew of detailed explanations of other topics.

This blogpost uses various images, the sources of which have been marked below them as a hyperlink. Care has been taken to use the images in accordance with the creator’s wishes whenever such information has been made available.

Finally, please keep in mind that I am by no means an expert on the subject, and you spot mistakes or think that some parts of this blogpost need to be adjusted, don’t hesitate to contact me at loshkin.oleg.95@gmail.com.

-----

<h1>Sharing Things</h1>
TODO: fun image

So far, multithreading has been presented to you in a very friendly manner: define a function, kick it off and <em>magic!</em>, everything works! This unfortunately is true only because so far, we didn't have to worry about sharing resources between multiple threads.

In our previous example of [approximating PI using a Monte Carlo method](), all that was needed was to generate a bunch of unrelated 2D points asynchronously and count how many of these landed inside the unit circle, an operation that could be split in a set of sub-operations that don't need to communicate with each other at all. But what if these sub-operations share some common counter? Or what if the order of these operations is important [(non-commutative)]()?

To explore these challenges, let's reformulate our initial problem a bit: instead of approximating PI, let's print it out, one digit at a time. Since PI is approximatively 3.141592 (rounded down for our purposes), if we wanted to print out it's digits sequentially, we would expect something like:

```
Pi is: 
3
1
4
1
5
9
2
```

Any other permutation of these numbers would be incorrect: we can no longer just kick off 7 threads simultaneously and ask them to print the digits of PI: we have made our problem order-dependant, each digit needs to come in order.

This also now implies that there needs to be some kind of shared counter that tells each running thread what is the next digit to print and each thread will need to have simultaneous access to it.

Let's illustrate the issue we will encounter if we're not careful with a somewhat absurd but representative analogy. Imagine 7 programmers sitting in a circle, each with a pen in one hand and a little card in the other. At the center of the circle is another small card with the number "0". All the programmers are tasked with incrementing the number on the small card in the middle of the circle by one if the number is 0 or positive or decrementing it if it's smaller than 0 by writing their result on the card in their hand and putting it on top of the one in the center. What happens next?

In practicality, what we would usually want is for the programmers to take a look at the number on the card in the middle of the circle, write the number that comes after the one they've just observed if it's a 0 and write it on their own little card and set it down in the center on top of the previous one. If each programmer does so in an orderly fashion, we should end up with a stack of 8 cards alternating between 0's and 1's, with a 0 at the top of the stack.
In reality of course, the programmers being very dense people and lacking the explicit instruction to wait their turn will simply all simultaneously look at the card in the middle with "0" on it, write "1" on their own cards, and all simultaneously pile up their cards in the middle on top of each other's in a disorderly fasion. We would therefore end up with a stack of 8 cards with the number "0" at the bottom followed by a bunch of cards with "1"'s and maybe a few "0"'s in the middle from the programmers that were daydreaming while the rest were executing the task asked of them, causing them to read the "1"'s set down by a quicker programmer.

This somewhat far-fetched analogy is exactly what happens when you're multithreading in C++ when you don't take care to instruct these "programmers" properly. You of course understand by now that the programmers in this analogy are threads and the little card in the middle in an address in memory. To be more technical, this is an example of a so-called [data race]().

TODO: illustration of a pile of cards with 0's and 1's.

-----

<h1>Getting Things in the Right Order</h1>
TODO: fun image

As programmers, we were probably taught that our code is executed in the order we wrote it. I'm sorry but have a harrowing revelation for you: <em>it isn't</em>.

This might indeed have been the case with early computers but CPU and compiler designers quickly realized that doing things strictly in order leads to the CPU just sitting there and waiting for needed data to arrive from the RAM. To avoid wasting this computing power, compilers and CPU are allowed to change the order of the execution of the snippets of your code as long as the outcome of this reordering results in an outcome identical to the one obtained had all the instructions been carried out sequentially <strong>on a single thread</strong>.

This is the crux of our headaches: the infrastructure we use has been initially created for single threaded programs, but with the development of multi-core hardware, we programmers now need to write software that will be ran on an unpredictable physical core (with it's personal cache memory), and in an unpredictable order!
Add to that the fact that modern OS's use [preemption](https://en.wikipedia.org/wiki/Preemption_(computing)) to implement multitasking, and you now also have to take into consideration that any thread that is running can be suddently interrupted by another, potentially one that operates on the very same memory, making the current thread's ongoing computation invalid, and the OS isn't going to check whether this is the case either!

How are we supposed to make anything work in such conditions? Well, we aren't, that's where multithreading synchronization primitives come into play.

Let's start with the simplest of synchronization primitives: [the mutex](https://en.wikipedia.org/wiki/Mutual_exclusion), first described by [Dijkstra](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra). A mutex (short for "mutual exclusion") is a high-level construct engineered and graciously provided to us mere mortals by the madmen that understand the arcane arts of hardware, compiler design, [memory barriers or fences](https://en.wikipedia.org/wiki/Barrier_(computer_science)) to give us the ability of making an arbitrary operation that <strong>shouldn't</strong> be interrupted (a [critical section](https://en.wikipedia.org/wiki/Critical_section)), actually <strong>unable to be interrupted</strong> (an [atomic operation](https://en.wikipedia.org/wiki/Atomicity_(database_systems))).

While a mutex ensures that critical operations stay atomic, they offer no way for us to order these operations, to do so, we need to use a [condition variable](). A condition variable is a terrible name for the concept of a having a set of threads wait for an arbitrary condition to become true, and when it does, one or all of them waking up and resuming work (perhaps a "thread waiting queue" may have been more descriptive).

Using these two concepts, one can create other multithreading synchronization tools, like the [semaphore](https://www.baeldung.com/cs/semaphore), which is a combination of a mutex, a condition variable and a counter.

Although I will not show how to use it in this blogpost, it can be convinient to know of the existance of the [std::atomic](https://en.cppreference.com/w/cpp/atomic/atomic) template as well which can be used in lieu of a mutex for simple data such as integers [(plain old data, POD)](https://en.wikipedia.org/wiki/Passive_data_structure).

-----

<h1>Running the Project</h1>
TODO: fun image

But that's enough theory for now, let's get our hands dirty! First, clone [the repository I've prepared for you](https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Producing_And_Consuming_PI):

https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Producing_And_Consuming_PI

Note that while this project compiles on Linux, easy_profiler has issues with tracking [context switch]() events on that platform, making the profiler's use limited on that platform.

If you're on Windows, just like last time, make an out-of-source build under "/build" using the CMake's GUI and generate a Visual Studio solution and open it after you've ran "moveDlls.bat" to move the dlls in the right place.

Once you've got the project running with the default CMake settings, you should get an output like so:

TODO: screen cap of console

-----

<h1>Let's Write some Code!</h1>
TODO: fun image

<h2>Singlethreaded Implementation</h2>

todo: sync not needed for BBP

<h2>Making Things Atomic</h2>

<h2>Putting some Order</h2>

-----

<h1>To Sum Things Up</h1>
TODO: fun image

<h2>There's Always More!</h2>

Flynn taxonomy
atomic, volatile, counting and binary semaphore, barriers, fences,
cooperative scheduling, fibers, coroutines, branch prediction, OS basics (interrupts, kernel calls, preemptive multitasking, kernel vs user space)
livelock, starvation, priority inversion, the dining philosphers
lock-free concurrency
spin-locks, reentrant locks

TODO: fun image