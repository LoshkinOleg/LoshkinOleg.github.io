---
title: "Multithreading in C++: Producing and Consuming PI"
date: 2022-09-10
layout: post
categories: blogpost
permalink: /:categories/:title
---

<br>
<img align="center" width="674" height="446" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/pie-eating-contest.jpg">
<em>[Picture of a pie eating contest](http://www.lauradays.org/pie-eating-contest.html)</em>
<br>
<br>

Hi, I’m Oleg, a Junior Game Audio Programmer and this is a continuation to [my previous blogpost on multithreading](). In the last post, I've shown you how to kick off threads and asynchronous tasks without getting into the details of memory sharing. Well, now's the time to get aquaited with the spooky side of multithreading!

In this blogpost, I'll introduce you to the concepts and challenges of resource sharing in a multi-threaded program and show you the basic C++11 facilities that are commonly used for writing portable multithreaded programs.

By the end of this blogpost, you'll hopefuly understand the bare minimum required to write multithreaded applications and will have a good list of learning references you can explore to learn more.

Note that in this blogpost, I will not focus on any hardware details relating to multithreading, but I encourage you to read on the subject since all of the constraints present when
solving multithreading challenges come from there. Once again, I recommend [Jason Gregory's "Game Engine Architecture"](https://www.gameenginebook.com/) for this.

-----

<br>

<h1>Credits</h1>

Before going any further, let me credit the people, software and learning resources I've used in the making of this blogpost:

The [practical project / exersice](https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Producing_And_Consuming_PI) I've put at your disposition could not have been possible without the following, in no particular order:
- [Visual Studio 2022 (windows)](https://visualstudio.microsoft.com/vs/) and [VSCode (linux)](https://code.visualstudio.com/)
- [Git](https://git-scm.com/) and [Github](https://github.com/)
- [CMake](https://cmake.org/)
- Sergey Yagovstev’s [easy_profiler](https://github.com/yse/easy_profiler)
- [Fabrice Bellard's implementation](https://bellard.org/pi/pi1.c) of the [Bailey-Borwein-Plouffe formula]() for generating digits of PI

Although not all explicitly mentioned in the content of this blogpost, the following learning resources were very useful in the making of this blogpost and I encourage you to read / watch them yourself:

- [www.modernescpp.com's explanation of Condition Variables](https://www.modernescpp.com/index.php/c-core-guidelines-be-aware-of-the-traps-of-condition-variables)
- My personal bible: [“Game Engine Architecture” by Jason Gregory](https://www.gameenginebook.com/). Most of the information presented in this blogpost can be found in more extensive details in that book alongside a slew of detailed explanations of other topics.

This blogpost uses various images, the sources of which have been marked below them as a hyperlink. Care has been taken to use the images in accordance with the creator’s wishes whenever such information has been made available.

Finally, please keep in mind that I am by no means an expert on the subject, and you spot mistakes or think that some parts of this blogpost need to be adjusted, don’t hesitate to contact me at loshkin.oleg.95@gmail.com.

-----

<h1>Sharing Things</h1>
<img align="center" width="553" height="320" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/sharingMeme.jpg">
<em>["Sharing is caring", generated with Imageflip Meme Generator](https://imgflip.com/i/2boi75)</em>

So far, multithreading has been presented to you in a very friendly manner: define a function, kick it off and <em>magic!</em>, everything works! This unfortunately is true
only because so far, we didn't have to worry about sharing resources between multiple threads.

In our previous example of [approximating PI using a Monte Carlo method](https://loshkinoleg.github.io/blogpost/Multithreading-in-Cplusplus-Approximating-Pi), all that was
needed was to generate a bunch of unrelated 2D points asynchronously and count how many of these landed inside the unit circle, an operation that could be split in a set of
sub-operations that don't need to communicate with each other at all. But what if these sub-operations share some common counter? Or what if the order of these operations is
important [(non-commutative)](https://en.wikipedia.org/wiki/Commutative_property)?

To explore these challenges, let's reformulate our initial problem a bit: instead of approximating PI, let's print it out, one digit at a time. Since PI is approximatively
3.141592 (rounded down for our purposes), if we wanted to print out it's digits sequentially, we would expect something like:

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

Any other permutation of these numbers would be incorrect: we can no longer just kick off 7 threads simultaneously and ask them to print the digits of PI:
we have made our problem order-dependant, each digit needs to come in order.

This also now implies that there needs to be some kind of shared counter that tells each running thread what is the next digit to print and each thread will need to have
simultaneous access to it.

Let's illustrate the issue we will encounter if we're not careful with a somewhat absurd but representative analogy. Imagine 7 programmers sitting in a circle, each with a
pen in one hand and a little card in the other. At the center of the circle is another small card with the number 0. All the programmers are tasked with incrementing the
number on the small card in the middle of the circle by one if the number is 0 or positive or decrementing it if it's smaller than 0 by writing their result on the card in
their hand and putting it on top of the one in the center. What happens next?

In practicality, what we would usually want is for the programmers to take a look at the number on the card in the middle of the circle, write the number that comes after the one
they've just observed if it's a 0 and write it on their own little card and set it down in the center on top of the previous one. If each programmer does so in an orderly fashion,
we should end up with a stack of 8 cards alternating between 0's and 1's, with a 1 at the top of the stack.

<img align="center" width="542" height="500" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/cards_noDataRace.png">

In reality of course, the programmers being very dense people and lacking the explicit instruction to wait their turn will simply all simultaneously look at the card in the
middle with "0" on it, write "1" on their own cards, and all simultaneously pile up their cards in the middle on top of each other's in a disorderly fasion. We would therefore
end up with a stack of 8 cards with the number "0" at the bottom followed by a bunch of cards with "1"'s and maybe a few "0"'s in the middle from the programmers that were
daydreaming while the rest were executing the task asked of them, causing them to read the "1"'s set down by a quicker programmer.

<img align="center" width="542" height="500" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/cards_dataRace.png">

This somewhat far-fetched analogy is exactly what happens when you're multithreading in C++ when you don't take care to instruct these "programmers" properly.
You of course understand by now that the programmers in this analogy are threads and the little card in the middle in an address in memory. To be more technical,
this is an example of a category of [race coditions called a data race](https://en.wikipedia.org/wiki/Race_condition).

-----

<br>

<h1>Getting Things in the Right Order</h1>
<img align="center" width="640" height="360" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/upsideDownHouse.jpg">
<em>["Device to Root Out Evil" by artist Dennis Oppenheim](https://www.easemytrip.com/blog/visit-some-unusual-structures-of-the-world)</em>

As programmers, we were probably taught that our code is executed in the order we wrote it. I'm sorry but have a harrowing revelation for you: <em>it isn't</em>.

This might indeed have been the case with early computers but CPU and compiler designers quickly realized that doing things strictly in order leads to the CPU just sitting there and
waiting for needed data to arrive from the RAM. To avoid wasting this computing power, [compilers and CPU are allowed to change the order of the execution](https://youtu.be/_qvOlL8nhN4)
of the snippets of your
code as long as the outcome of this reordering results in an outcome identical to the one obtained had all the instructions been carried out sequentially
<strong>on a single thread</strong>.

This is the crux of our headaches: the infrastructure we use has been created for single threaded programs, but with the development of multi-core hardware, we programmers
now need to write software that will be ran on an unpredictable physical core (with it's [personal cache memory](https://en.wikipedia.org/wiki/CPU_cache)),
and in an unpredictable order!
Add to that the fact that modern OS's use [preemption](https://en.wikipedia.org/wiki/Preemption_(computing)) to implement multitasking, and you now also have to take into
consideration that any thread that is running can be suddently interrupted by another, potentially one that operates on the very same memory, making the current thread's ongoing
computation invalid, and the OS isn't going to check whether this is the case either!

How are we supposed to make anything work in such conditions? Well, luckily we aren't left to our own devices, that's where multithreading synchronization primitives come into play!

Let's start with the simplest of synchronization primitives: [the mutex](https://en.wikipedia.org/wiki/Mutual_exclusion), first described by
[Dijkstra](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra). A mutex (short for "mutual exclusion") is a high-level construct engineered and graciously provided to us mere
mortals by the grand wizards that understand the arcane arts of hardware, compiler design, [memory barriers or fences](https://en.wikipedia.org/wiki/Barrier_(computer_science))
to give us the ability of making an arbitrary operation that <strong>shouldn't</strong> be interrupted (a [critical section](https://en.wikipedia.org/wiki/Critical_section)),
actually <strong>unable to be interrupted</strong> (an [atomic operation](https://en.wikipedia.org/wiki/Atomicity_(database_systems))).

While a mutex ensures that critical operations stay atomic, they offer no way for us to order these operations.
To do so, we need to use a [condition variable](https://en.wikipedia.org/wiki/Monitor_(synchronization)).
A condition variable is a terrible name for the concept of a having a set of threads wait for an arbitrary condition to become true, and when it does,
one or all of them waking up and resuming work (perhaps a "thread waiting queue" may have been more descriptive).

Using these two concepts, one can create other more complex multithreading synchronization tools such as the [semaphore](https://www.baeldung.com/cs/semaphore),
which can be implemented as a combination of a mutex, a condition variable and a counter (note that since C++20, there exists the
[std::counting_semaphore](https://en.cppreference.com/w/cpp/thread/counting_semaphore) in the C++ language).

Although I will not show how to use it in this blogpost, it can be useful to know of the existance of the [std::atomic](https://en.cppreference.com/w/cpp/atomic/atomic) template
as well which can be used in lieu of a mutex for simple data such as integers [(plain old data, POD)](https://en.wikipedia.org/wiki/Passive_data_structure). As it's name indicates,
any operation on a simple data variable wrapped by this template becomes atomic.

-----

<h1>Running the Project</h1>
TODO: fun image

But that's enough theory for now, let's get our hands dirty! First, clone [the repository I've prepared for you](https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Producing_And_Consuming_PI):

https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Producing_And_Consuming_PI

Note that while this project compiles on Linux, easy_profiler has issues with tracking [context switching](https://en.wikipedia.org/wiki/Context_switch)
events on that platform, making the profiler's use limited on that platform.

If you're on Windows, just like last time, make an out-of-source build under "/build" using the CMake's GUI and generate a Visual Studio solution and open it after you've ran "moveDlls.bat" to move the dlls in the right place.

Once you've got the project running with the default CMake settings, you should get an output like so:

<img align="center" width="616" height="396" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/projSetup0.png">

Let me explain this output real quick. You'll be implementing four (or three, really, you'll see) ways to [produce and consume](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)
individual digits of PI: the "Producer" functions will generate digits of PI at a given position and identify themselves and "Consumer" functions will
also identify themselves and append all of these things to a string output.
In the last line of the example above for instance, you can see that a Consumer with id "5" has recieved from Producer with id "4" a PI digit of "2".

To make more sense of this, let's take a look at the contents of the "/Application/src/main.cpp" file. The first lines you should already be familiar with, they define
whether to run the implementation that I've provided ("/Application/include/workingImplementation.h") or your own implementation ("/Application/include/exercise.h").
Next is a threads pool and an array defining the indices of the PI digits to Produce:

<img align="center" width="621" height="212" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/projSetup1.png">

Next is a function that will reset all the variables between runs of different producing and consuming approaches. The variables it resets that you don't see in
this current file come from the implementation headers. The function just resets variables to their default values and randomizes <em>iterations</em>, the order in which
the digits of PI will be Produced:

<img align="center" width="616" height="161" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/projSetup2.png">

This might seem like an odd way to structure things but you'll see why we're doing things this way below: it's to better demonstrate the properties of std::mutex
and std::condition_variable.

Finally comes the main function, and you're already familiar with the start and the end of the function: it's just some easy_profiler code that initializes the profiler
and writes the results to disk. In between, you've got calls to the functions you'll be implementing:

<img align="center" width="629" height="218" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/projSetup3.png">

For our purposes, we'll be Producing and Consuming the first 7 digits of PI, but the order of PI digits will be random (thanks to the use of <em>iterations</em>).
This is done to emphasize that while some code like this might coincidentally work as expected:

```
for (size_t i = 0; i <= LAST_DIGIT; i++)
{
	threads.push_back(std::thread(NoMutex_Producer, i));
	threads.push_back(std::thread(NoMutex_Consumer, i));
}
for (auto& thread : threads)
{
	thread.join();
}
```

It is <strong>not guaranteed</strong> to work. Explicitly shuffling the order of the push_back's like we're doing here makes this point obvious.

Now that you've seen how the main logic of the Application works, go ahead and uncheck the USE_WORKING_IMPLEMENTATION define in CMake GUI and let's take a look at
the part of the code you'll actually be implementing, which is contained within "/Application/include/exercise.h":

<img align="center" width="594" height="629" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/projSetup4.png">

First, the included header "/Application/digitsOfPi.h" contains a modified implementation of [Fabrice Bellard's implementation of the Bailey-Borwein-Plouffe formula](https://bellard.org/pi/pi1.c)
used here to compute an arbitrary digit of PI. The details of this implementation are out of the scope of this blogpost (as well as out of my own understanding to be honest)
but to sum it up: it's a very efficient algorithm allowing to compute an arbitrary set of digits of PI without having to compute all the part of PI that comes before
the set of digits we're interested in.

Next are some literals if you wish to change the range of PI to compute, I've chosen the first 7 digits because they're the ones we're most familiar with but the code
should work just fine with any other positive values.

The PieceOfPi struct is the data that Producers and Consumers will be passing to each other and it contains fields for the actual digit of PI and two fields allowing
identification of the Consumers and Producers involved in a given transaction. It also has a method to be able to conveniently print it.

Next up is the MessWithCompiler function which, as it's name indicates, makes the code from which it is called from unpredictable both for the compiler and the CPU:
it introduces a random delay in the execution that varies between 0 and 25 milliseconds. We'll be using this function to ensure that our code is capable to withstand
unpredictable delays and unpredictable order of execution of threads.

Finally, we've got the first three global variables we'll be using to implement the regular, non-multithreaded approach of our producer-consumer problem.
The <em>buffer</em> variable is the unit of PI that will be passed from Producer to Consumer at a time, <em>toPrint</em> is a string that the Consumers will append
their results to and <em>iteration</em> will be used to define the next digit of PI to Produce.

-----

<h1>Let's Write some Code!</h1>
TODO: fun image

Alright, enough theory, let's go to coding stuff! Just like in the previous blogpost, I've marked the areas where you should be writing code with descriptive TODO's,
so feel free to attempt all of those things by yourself first and come back to this blogpost if you're stuck!

<h2>SingleThreaded and NoMutex Implementations</h2>

Let's start with a SingleThreaded, sequential implementation. I doubt you'll have any trouble implementing this first approach, as a reminder, this is how
SingleThreaded_Producer and SingleThreaded_Consumer functions are used in main:

```
Reset();
std::cout << "Using SingleThreaded functions to generate digits of PI..." << std::endl;
for (size_t i = FIRST_DIGIT; i <= LAST_DIGIT; i++)
{
	SingleThreaded_Producer(i);
	SingleThreaded_Consumer(i);
}
std::cout << toPrint << std::endl;
```

Once implemented, the two functions should look like this:

<img align="center" width="586" height="288" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/singleThreaded0.png">

And if you run the Application, you should have the following output:

<img align="center" width="472" height="159" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/singleThreaded1.png">

At this point, you could insert some MessWithCompiler calls if you want to ensure that this implementation will work no matter what: it is all executed sequentially,
so any unpredictable delays will have no effect on the result. You can see for yourself that right now, everything is done on a single thread and sequentially using
easy_profiler's GUI:

<img align="center" width="616" height="23" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/singleThreaded2.png">

Now that we've got our SingleThreaded implementation working, let's see what happens if we try to do the same exact thing, but this time concurrently with 14 threads,
one to Produce and one to Consume a PieceOfPi, per digit. Let's just copy paste the same code in the NoMutex functions at their appropriate places:

<img align="center" width="651" height="269" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/noMutex0.png">

Uh-oh! This time, I didn't even have to uncomment the MessWithCompiler call to force a data race I had left in case things accidentally worked as intended!:

<img align="center" width="418" height="162" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/noMutex1.png">

So what's going on here? Let's see if we can get any clues by looking at the results of the profiler (I did introduce an artificial 1 ms extra delay in the functions
just so that we could better see what's going on):

<img align="center" width="617" height="358" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/noMutex2.png">

Ah. Without a way to make operations [atomic](https://en.wikipedia.org/wiki/Atomicity_(programming)), meaning <strong>unable to be interrupted</strong>.
Right now, all of our threads are technically allowed to execute the same code all at the same time, using shared resources (<em>buffer</em>, <em>toPrint</em>
and <em>iteration</em>). Clearly, this is a [critical](https://en.wikipedia.org/wiki/Critical_section) operation, meaning one that <em>should not be interrupted</em>!
In our case, since we're not protecting our shared resources, we end up with all threads running simultaneously, reading and writing to the shared resources unchecked!

<h2>Making Things Atomic</h2>

Well, let's try to remedy to this situation then: let's use a [std::mutex](https://en.cppreference.com/w/cpp/thread/mutex)! I've already defined a global mutex
named <em>m</em> for you, all you have to do is use it in the MutexOnly functions <strong>before</strong> operating on whatever resources you're trying to protect.
To do so, you have to "lock" the mutex using one of the [C++ locks](https://en.cppreference.com/w/cpp/thread).
To know what lock you need to use, here's a brief introduction of the most useful C++ locks:
- The [unique lock](https://en.cppreference.com/w/cpp/thread/unique_lock) is a very generic kind of mutex lock that can be [.lock()](https://en.cppreference.com/w/cpp/thread/unique_lock/lock)'ed
and [.unlock()](https://en.cppreference.com/w/cpp/thread/unique_lock/unlock)'ed any number of times, though usually you wouldn't do that yourself, the standard does that
for you. You should only use this kind of lock when it is required, when using std::condition_variable's for instance.
- The [lock_guard](https://en.cppreference.com/w/cpp/thread/lock_guard) and it's successor the [scoped_lock](https://en.cppreference.com/w/cpp/thread/scoped_lock) do
very similar things: they both <strong>lock once</strong> at construction and <strong>unlock once</strong> at destruction. This should make them your go-to locks since
locking a mutex involves a [kernel call](https://en.wikipedia.org/wiki/System_call), which is relatively expensive. The distinction between the two locks is that the
lock_guard is only capable of locking a single mutex at a time (leaving the possibility of a [deadlock](https://en.wikipedia.org/wiki/Deadlock)) whereas a
scoped_lock can safely lock multiple mutexes if needed. So between the two, use the scoped_lock unless you need a lock_guard for some legacy reasons.
- Finally, the [shared_lock](https://en.cppreference.com/w/cpp/thread/shared_lock) is a very useful type of lock because it does not actually completely immobilise
the protected shared resource: a mutex locked by a shared_lock can still be used by other threads as long as all users of the mutex are <em>reading</em> from the protected
resource and not modifying it. You can think of the shared_lock as a "read-only lock" if you wish, as long as you're not modifying anything, locking a shared_lock
does not involve an expensive kernel call. Use shared_locks when you only need to read from a shared resource.

The useful thing about mutexes is that as their name indicates, locking them is mutually exclusive: <strong>only one</strong> thread can ever lock a mutex, and therefore
it's associated resources, at a time, making whatever operations you might perform on these shared resources atomic.

Keep in mind that when using the standard C++ locks, whenever you instanciate a lock it is automatically .lock()'ed unless you specify otherwise, as can be done with
unique locks. The opposite is also true, by default, when a lock goes out of scope, it's destructor unlocks the mutex.
I will not explain those in depth here, but you should also be aware that not all mutex locking operations are [blocking](https://en.wikipedia.org/wiki/Blocking_(computing)),
unique_lock's and shared_lock's also have [.try_lock()](https://en.cppreference.com/w/cpp/thread/unique_lock/try_lock) methods (and its variations) which can be used for
[polling](https://en.wikipedia.org/wiki/Polling_(computer_science)) based locking, but you can still end up with a [livelock](https://en.wikipedia.org/wiki/Deadlock)
instead of a deadlock.

But what does all of this mean for our code? Well that we'll be using a scoped_lock in both the Producer and Consumer functions (the consumer modifies <em>buffer</em> and <em>toPrint</em>
so we can't use a shared_lock there unfortunately!):

<img align="center" width="576" height="438" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/mutexOnly0.png">

Let's run it and see what happens!:

<img align="center" width="439" height="154" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/mutexOnly1.png">

That's still not right. Let's take a look at the outputs of the profiler:

<img align="center" width="657" height="346" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/mutexOnly2.png">

While we did make our operations atomic by using the mutex, the operations are still executed in the wrong order! The very first function to execute is a Consumer, which
understandably has nothing to consume, resulting in the "digit: " output and if you look at the two last functions to get called, they're both Producers!
This is because a mutex has only one effect on the order of operations: it makes the operations <em>sequential</em> and that is it. For our PI producing and consuming to work
we need one more additional tool.

<h2>Putting some Order</h2>
TODO: fun image

So, what is our secret weapon that will allow us to organize this unpredictable order of execution into something orderly you might ask? Introducing: the
[std::condition_variable](https://en.cppreference.com/w/cpp/thread/condition_variable)!
A condition_variable is a construct that allows different threads to notify each other that <em>something</em> has happened. A condition_variable works alongside a
mutex to put a thread to sleep (a.k.a. to block it) but instead of being woken up whenever the locked resource becomes available again, it is woken up upon receiving
a notification via the condition_variable.

In C++, you use them like so:

```
std::unique_lock<std::mutex> lck(m); // Mutex is locked in the constructor here.
cv.wait(lck, []{ return somePredicate; }); // Mutex is implicitly unlocked by the compiler here.
// Mutex is implicitly locked again by the complier here.
DoThings();
```

And some other thread will be responsible for waking up the sleeping thread by calling the following:
```
cv.notify_one(); // Or cv.notify_all() to wake up every thread .wait()'ing on the cv
```

Note that while it is technically not required to provide a predicate (some condition to check) to the .wait() method, practically, doing so is mandatory due to
[spurious wakeups](https://en.wikipedia.org/wiki/Spurious_wakeup):
your thread might wake up unexpectedly when they shouldn't due to some quirks of the OS, so if they do, providing them with a way to check whether they
should actually
continue execution or not is pretty much mandatory.
Note also that this is pretty much the only case when it's okay to call .wait() while holding a lock because the condition_variable locks/unlocks the mutex
it's using automatically, doing so in most other situations would result in a deadlock.

We'll be using two condition_variable's since we want the Producers to wake up Consumers and vice-versa, if we used only one condtion_variable, we couldn't specify
whether a Producer or a Consumer should be woken up. So with all of this said, this is what the code final working PI producing and consuming code looks like!
(despite our best efforts to break it):

<img align="center" width="582" height="879" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/cv0.png">

And this results in the correct series of digits as we can see:

<img align="center" width="381" height="154" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/cv1.png">

We can confirm this by looking at the results of the easy_profiler:

<img align="center" width="709" height="268" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/cv2.png">

-----

<h1>There's Always More!</h1>
TODO: fun image

As much as I would have liked to tell you that this is all you need to know to write good multithreaded code, that's unfortunately not the case. What I've presented
here barely scratches the surface of the complexities of making multithreaded programs work but I hope that with these two blogposts you now have the bare essentials
to make a multithreaded program and more importantly, that you've got now some references you can follow up on to learn more.

If you want one more little exercise to understand the difference between concurrency and parallelism and it's applications, I invite you to answer a question
for yourself: can you make the code we've just written [lock-free](https://en.wikipedia.org/wiki/Non-blocking_algorithm) (fancy way of saying without using
mutexes and locks)? Could you still do it if instead of using [Bailey-Borwein-Plouffe](https://en.wikipedia.org/wiki/Bailey%E2%80%93Borwein%E2%80%93Plouffe_formula)'s formula for computing PI,
we had used another method for computing PI, such as the [Leibniz formula for PI](https://en.wikipedia.org/wiki/Madhava-Leibniz_series)?

I hope this blogpost has taught you something new and useful and stay curious!

TODO: fun image