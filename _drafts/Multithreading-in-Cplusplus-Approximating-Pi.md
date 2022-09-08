---
title: "Multithreading in C++: Approximating PI"
date: 2022-09-07
layout: post
categories: blogpost
permalink: /:categories/:title
---
<br>
<img align="center" width="675" height="488" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/catYarn.jpg">
<em>["She didn't anticipate the yarn fighting back"](https://www.reddit.com/r/aww/comments/8ush6f/she_didnt_anticipate_the_yarn_fighting_back/)
<br>
<br>
Hi, I'm Oleg, I'm a freshly graduated Audio Games Programming Bachelor and while reviewing C++ multithreading and multiprocessing,
I've decided to share with you this little project that introduces you to the basics of multithreading and multiprocesing in C++,
or more specifically to parallel processing.<br>

This a very important part of any modern programmer's job, be it performance critical or not: contrary to what
some might initially assume when first introduced to the concept, multithreading is not used only to make
your code run faster (if done inappropriately, more often than not it will worsen the performance of your
application instead!).

This is why this blogpost exists, to hopefully help you understand the subject better and when it does and
does not make sense to take advantage of a multithreaded and multiprocessing architecture!

<h1>
  Credits
</h1>
Before going any further, let me thank and credit the people whose software and learning resources I've used in the making
of this blogpost:

The [practical project / exercise](https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Approximating_Pi) I've put at your disposition could not have been possible without the following,
in no particular order:
- [Visual Studio 2022](https://visualstudio.microsoft.com/vs/)
- [Git](https://git-scm.com/) and [Github](https://github.com/)
- [CMake](https://cmake.org/)
- Sergey Yagovstev's very useful [easy_profiler](https://github.com/yse/easy_profiler)

Although not all necessarily mentioned in the text, the following learning resources were very useful in the
making of this blogpost and I encourage you to read them yourself:
- [Estimating the value of Pi using Monte Carlo, GeeksforGeeks, updated on July 2022](https://www.geeksforgeeks.org/estimating-value-pi-using-monte-carlo/)
- [CppCon 2018 Jefferson Amstutz Compute More in Less Time Using C++ Simd Wrapper Libraries](https://youtu.be/8khWb-Bhhvs)

Finally, this blogpost uses various images, the sources of which have been marked below them.
Care has been taken to use the images in accordance with the creator's wishes whenever such information
has been made available (no explicitly copyrighted images and the like).

<h1>
  Definitions (they're handy!)
</h1>
<h2>
  Multithreading
</h2>

For those that are somehow unaware of the concept, multithreading is:
<br>
<br>
<p><em>
     "[...] the ability of a central processing unit (CPU) (or a single core in a multi-core processor) to provide multiple threads of execution concurrently, supported by the operating system."<br>
   </em></p>
<em>- [Multithreading (computer architecture), Wikipedia, 2022](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture))
<br>
<br>

In simpler terms, it is the basic technique that allows computers to run many programs at once, something that
was not possible with earlier computers: early home computers such as the [Commodore 64](https://en.wikipedia.org/wiki/Commodore_64) were incapable
of true multithreading, making the computer able to run only one program at a time [(though later it would become
capable of](https://wiki.c2.com/?CommodoreSixtyFour) [multi-tasking](https://en.wikipedia.org/wiki/Computer_multitasking) [with the GEOS](https://en.wikipedia.org/wiki/GEOS_(8-bit_operating_system)))].
<br>
<br>
<img align="center" width="349" height="300" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/c64Computer.jpg">
<em>[Image of a Commodore 64](https://history-computer.com/commodore-64-guide/)
<br>
<br>

Running multiple programs simultaneously can be achieved in various ways, and not all of them require a multi-core processor.
Your computer for instance is currently running many more programs that your machine has cores as you can see if you
open up your task manager:

<img align="center" width="674" height="620" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/taskManager.png">
<br>
This is because the OS is using what is called [timeslicing](https://en.wikipedia.org/wiki/Preemption_(computing)) to let every program a little bit
of time on the CPU to execute partially before letting another program run.

<h2>
  Multiprocessing
</h2>

In 2001, IBM released the [POWER4](https://en.wikipedia.org/wiki/POWER4), the first multi-core CPU as one thinks
of one today. It ran at 1.3 GHz and had two cores on the same CPU die:

<img align="center" width="612" height="601" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/power4Die.png">
<em>[Die of the POWER4 CPU](https://nanopdf.com/download/iii-multicore-processors-5_pdf)

This is very different from multi-tasking: with a multi-core machine, it can physically run multiple programs
trully at the same time without stopping and starting another to do so. With this new advancement, it has become
possible to take advantage of a multi-core CPU to speed up complex computations if done correctly.

Indeed, since a CPU nowadays features many cores, we can naively understand that running our program on a single
core of our 4-core CPU would theorically result in 1/4 of the potential performance (which is a VERY gross oversimplification
by the way, in reality it's not the case and there's many factors that come into play, to the point that it's best
to write a single-threaded program unless there's a good reason to do otherwise).

Finally, we currently see the exciting emergence of [general-purpose GPU computing (GPGPU)](https://en.wikipedia.org/wiki/General-purpose_computing_on_graphics_processing_units)
which consists in taking advantage of the hugely parallel computing pipelines of GPUs to greatly accelerate the computing
of non-graphics specific tasks: [SteamAudio for instance can use AMD's GPU's to accelerate it's real-time sound
simulations](https://gpuopen.com/trueaudio-next-now-integrated-steam-audio/) and the [Playstation 4 has specifically
been produced with a more perfomant than strictly required GPU](https://youtu.be/f8XdvIO8JxE?t=2556) to allow games to use the console's GPU for
non-graphics related computations.<br>
General-purpose GPU computing however is well beyond the scope of this blogpost.

<img align="center" width="531" height="535" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/gt200GpuDie.gif">
<em>[Die of the Nividia's GT200 GPU](https://www.realworldtech.com/gt200/11/)

<h2>
  Concurrency vs. Parallelism
</h2>

<img align="center" width="680" height="333" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/parallelismVsConcurrency.jpg">
<em>Illustration of parallelism vs. concurrency with images taken from ["How to Cut Green Onions"](https://homecookbasics.com/how-to-cut-green-onions/) and [www.dreamstime.com](https://www.dreamstime.com/tired-housewife-carrying-lots-heavy-pots-saucepans-cutting-boards-kitchen-cooking-food-stuff-frail-slim-young-woman-image209469480#_) respecively.

Concurrency and parallelism can sound like synonyms but they are not. Rob Pike, the creator of the [Go programming language](https://en.wikipedia.org/wiki/Go_(programming_language))
described the concepts very elegantly in [one of his talks](https://go.dev/talks/2012/waza.slide#8):
<p><em>
  "Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things once."<br>
  </em></p>

To give a more concrete and relatable example, concurrency is like cooking a stew: you might start by cutting some beef
and while it's sizzling in a pan pour some water in a pot and set it to boil. While that's going on, you'd chop up
some vegetables and at some point throw all of that good stuff in the boiling water and maybe add some flour.
You might then after a while put the pot in the oven for a few hours and in the meantime you'd make some potatoes and some homemade sauce.
This is concurrency: doing lots of things at once.

Parallelism is much simpler, it's like chopping up some scallions: you'd take a whole bunch of them and chop them
all up at once, each motion of the knife cutting the whole bunch at once.

-----

<h1>
  Approximating PI
</h1>
<img align="center" width="640" height="640" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/piPie.jpg">
<em>Image of a PI pie from a [recipe by Jules Douglass](https://thefeedfeed.com/julesfood/cherry-pi-pie).


To get familiar with multithreading and parallelism in C++, we'll be approximating PI using the [Monte Carlo method](https://en.wikipedia.org/wiki/Monte_Carlo_method)
inspired by [GeeksForGeeks' tutorial](https://www.geeksforgeeks.org/estimating-value-pi-using-monte-carlo/).
"Monte Carlo analysis" is essentially a fancy way of saying that we'll be using randomness to progressively
approximate the deterministic value that is PI.

The idea is fairly simple: generate a whole lot of [uniformly distributed](https://en.wikipedia.org/wiki/Continuous_uniform_distribution)
random 2D points with component values in the range [-1;1] and this gives you points that lie within the [unit square](https://en.wikipedia.org/wiki/Unit_square).
You then check how many of these points lie inside the [unit circle](https://en.wikipedia.org/wiki/Unit_circle) that in inscribed in the unit square. Since we know how to calculate the areas of a square and a circle, we can get a ratio of their areas:

```
Area of circle = PI * radius * radius
```
and
```
Area of square = side * side = (2 * radius) * (2 * radius) = 4 * radius * radius
```
So we'd write the ratio of the two as:
```
(PI * radius * radius) / (4 * radius * radius)
```
If we then simplify the expression, we end up with this equation:
```
Area of circle / Area of square = PI / 4
```

<strong>TODO: insert image of unit circle and square</strong>

Since we're generating A LOT of points, as the number of random points grows,
we can approximate the above equation instead with the following one:
```
Number of points inside the circle / Number of points inside the square = PI / 4
```
Flip the equation around and we end up with:
```
PI = 4 * Number of points inside the circle / Number of points inside the square
```

As the number of random points we're counting grows, using this equation we'll be getting progressively closer to
the true value of PI, theorically reaching it if we somehow managed to generate an infinite amount of
random, uniformly distributed points.

<h2>
  But why PI?
</h2>
<img align="center" width="561" height="422" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/eatenTooMuch.jpg">
<em>Poor [Pingu](https://en.wikipedia.org/wiki/Pingu) has eaten too much, taken from [www.sharecopia.com](https://www.sharecopia.com/ve-eaten-order-dessert/)

Approximating PI with the Monte Carlo method is a classic example of what is known as an [embarrasingly parallel problem](https://en.wikipedia.org/wiki/Embarrassingly_parallel):
it approximates the value of PI by running a very simple procedure many times over and each step of the
iteration is completely independant from the previous.
This makes this problem particualrly well suited to introduce you to the parallel computing tools of C++: it
(ideally but not actually) means that there is no need to bother ourselves with the challenges of [memory sharing](https://en.wikipedia.org/wiki/Shared_memory) and
with [race conditions](https://en.wikipedia.org/wiki/Race_condition).<br>
That being said, as you'll see in a follow-up blogpost to this one, even our simple problem is affected by
those factors, but for now and for the sake of simplicity, we'll just to turn the blind eye to those issues. One step at a time!

-----

<h1>
  Running the project
</h1>

But enough chatting, let's download and setup the project we'll be working in. First, clone the [Github repository
I've prepared for you](https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Approximating_Pi): https://github.com/LoshkinOleg/Multithreading_In_Cplusplus_Approximating_Pi 

Once that's done, make an [out-of-source](https://cprieto.com/posts/2016/10/cmake-out-of-source-build.html) CMake build
using CMake's GUI application (or using the command line if you're feeling like impressing people that don't know any better):

<img align="center" width="727" height="1111" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/outOfSourceBuild.png">

For now, leave all the options at default, I'll just show you around before starting coding.
Once you've made an out-of-source CMake build under "/build" (it's important to name the folder precisely like so for relative paths to work correctly),
open up the VS solution and set the "Application" project as the Startup one:

<img align="center" width="612" height="423" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/startupProject.png">

Now, go ahead and compile and launch the application, you should end up with an error message about a missing .dll:

<img align="center" width="536" height="197" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/missingDll.png">

This is because we'll be using Sergey Yagovstev's [easy_profiler](https://github.com/yse/easy_profiler) to profile our code
to see whether we're actually doing things in parallel and whether our parallel computing is actually more performant than
our baseline single-threaded implementation.
I've saved you the trouble of compiling the profiler dll yourself and have put everything necessary under "thirdparty/easy_profiler/".
You'll find the application we'll be using to visualize the results of the profiler under "/thirdparty/easy_profiler/bin/profilerApp/profiler_gui.exe", but I'll introduce you to it a bit later.

To fix this missing .dll error, if you're on Windows, just run the "moveDlls.bat" script I've provided for you at the root of the repository.
If you can't run it, just manually copy the "/thirdparty/easy_profiler/bin/easy_profiler.dll" to "/build/Application/bin/Debug/" and "/build/Application/bin/Release/", which
are empty folders that should exist if you've correctly done the out-of-source CMake build.

Now that it's done, when you launch the application this time, you should have some approximations of PI being output to the console window:

<img align="center" width="572" height="122" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/workingApproxOfPi.png">

The exact approximations of PI will vary but should all be around 3.14. But let me show you around!<br>
If you open up "/Application/src/main.cpp" you'll see the code that's run every time you launch the application,
regardless whether you're running my working implementation or running your own implementation.
At the top of the file, you've got the USE_WORKING_IMPLEMENTATION define from before being used to include either
"/Application/include/workingImplementation.h" which is a header that implements the PI approximating functions whose result you've seen earlier
or to include your own "/Application/include/exercise.h" header where you'll be writing your own implementation:

<img align="center" width="382" height="282" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/implementationSwitch.png">

In the body of the main() function, you'll find some literals of relevance to you: ITERATIONS determines how many points we'll be using to approximate PI. Add or remove a few 0's
depending on the performance of your machine, the idea is for the algorithms to take a few seconds to execute so that we can clearly see it in the easy_profiler's GUI but it really
doesn't need to be taking a long time to compute every time you launch the application to see if things are working, we're not actually
trying to approximate PI for any practical use.

Next, NR_OF_WORKERS defines into how many parts we'll be chopping up the workload into when we'll be writing the parallel versions
of our PI approximating functions. This number should be at the number of logical cores your CPU has - 2.
You can find the number of logical cores your CPU has in the Task Manager if you're running Windows:

<img align="center" width="496" height="441" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/numberOfCores.png">

Note that it's the <strong>"logical</strong> cores" you're interested in, not just "cores", if you're interested in the distinction,
you can read up on it [here](https://unix.stackexchange.com/questions/88283/so-what-are-logical-cpu-cores-as-opposed-to-physical-cpu-cores).

The reason I'm suggesting to set NR_OF_WORKERS to logical cores - 2 is mostly for ease of readability of the profiler's results: one core would be reserved for running
the main thread (the main() function) and the others for running the parallelized PI approximating algorithm. I'm suggesting -2 rather than -1 just
to ensure that the work can be split very evenly for demonstration's sake: if the number of worker threads is even, dividing ITERATIONS by NR_OF_WORKERS will always
end up in a perfectly even workload split between the cores and more generally will make debugging and wrapping your head around what's going on
just a tid bit easier.

What follows these literals is just some code to actually execute the PI approximating functions and to time them.
You've already seen that in addition to outputting the approximation, this program also outputs the number of system clock ticks the algorithm has taken to execute.

Finally, if you've left BUILD_WITH_EASY_PROFILER enabled in CMake's GUI, the last bit of code just writes the profiling data to
"/build/profilerOutputs/session.prof". We'll see later how to use that data to get an idea of what's going on.

Now that we've made sure everything works as indended, go back to CMake's GUI application and uncheck "USE_WORKING_IMPLEMENTATION" checkbox. This will make the
code we just saw use "/Application/include/exercise.h" to approximate PI and if you now run the program again you'll see that all the
approximations will be of 0 since it's not yet implemented.

-----

<h1>
  Let's get to implementing!
</h1>
<img align="center" width="475" height="322" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/typing.gif">
<em>[Typing furiously, taken from GIFER](https://gifer.com/en/6nVp)
<h2>
  SingleThread()
</h2>

Enough of the boring stuff, let's get to writing some code! From now on, I'll explain step by step how to first implement a
single-threaded PI-approximating function and then show you how you can take advantage of the highly parallel
nature of the problem to accelerate it's computation by taking advantage of a multi-core modern CPU using
only the tools provided by the standard C++ library (C++20 standard in our case).

I highly encourage you to attempt this by yourself at first and only come back to this blogpost if you're stumped
or need a hint for how to approach the next part of the task you're trying to do.

Go ahead and open up "/Application/include/exercise.h". Inside, you'll find the four partially implemented functions we'll be implementing in order:
- SingleThread() will simply approximate PI entirely on the main thread, we'll be using it compare our other approaches against.
- SimpleAsync() will be our first attempt at using std::async and stde::future to try to parallelize the algorithm implemented in SingleThread().
- AsyncNoRef() will be a variation on the previous function. You'll see the details later how it differs from the previous function.
- Threads() will use std::thread, std::future and std::promise to implement the same algorithm without using std::async.

Let's start by implementing the simple single threaded version of the algorithm in SingleThread():
<img align="center" width="649" height="170" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/singleThread0.PNG">

I've maked where you should write your code with the TODO comment. Before it, you'll find the random number generator <em>e</em> and the uniform floats distribution <em>d</em> ranging between
[-1;1[ ([1.0f is exclusive due to some quirk of the standard library](https://en.cppreference.com/w/cpp/numeric/random/uniform_real_distribution) but since we're working with floats, it
shouldn't be much of a problem). Note that the random generator isn't [seeded](https://stats.stackexchange.com/questions/354373/what-exactly-is-a-seed-in-a-random-number-generator) for the purposes of this blogpost (but you can trying to seed it and see what happens! The issues you'll
observe will be the subject of the next blogpost).
There's also the <em>insideCircle</em> variable which we'll use to count the number of random points inside the unit circle. And at the bottom, of course, there's the return statement.

Let's get to it then! The first step is simple enough: all we need is to generate a 2D point defined by two components <em>x</em> and <em>y</em> using the random number generator
and check if it is inside the unit circle by computing the magnitude of the vector going from the origin to the point. Since the origin is by definition at (0;0), we can directly pass
<em>x</em> and <em>y</em> to the Magnitude() function I've prepared for you.
If the point is inside the circle, meaning if the magnitude is <= 1.0f, we'll increment the <em>insideCircle</em> counter.
<img align="center" width="313" height="199" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/singleThread1.PNG">

Very difficult, isn't it? All that's left is to apply the formula discussed earlier to approximate PI. Here we can use <em>iterations</em> in place of "Number of points inside the square"
since any point resulting from a [1;1] ranging distribution will necessarily be part of a unit square:
```
PI = 4 * Number of points inside the circle / Number of points inside the square
```
<img align="center" width="426" height="23" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/singleThread2.PNG">

And that's it for the SingleThread() function, really:
<img align="center" width="486" height="100" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/singleThread3.PNG">

Before we move on to implementing the other functions, let's briefly get familiar with easy_profiler's GUI application. Every time you run the program, it will output
profiling data to "/build/profilerOutputs/session.prof". Let's take a look at it in easy_profiler's GUI application (if you try reading it with a text editor, you'll see that it's
all binary data, so you need to use the GUI application to make sense of it.)
Navigate to and run "/thirdparty/easy_profiler/bin/profilerApp/profiler_gui.exe" and open up "/build/profilerOutputs/session.prof" by clicking on the folder icon at the top-left of the
window:
<img align="center" width="638" height="502" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/singleThread4.PNG">

Once it's opened, you'll be able to navigate in the timeline and see the little profiling blocks that were defined in the code using EASY_BLOCK macros.
For now, as you can see, we've only got one thread running and if you mouse over the SingleThread profiling block, you'll see the time it has taken to execute our newly implemented function:
<img align="center" width="677" height="337" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/singleThread5.PNG">

When we'll start kicking off (meaning starting) new threads in the following functions, you'll see more threads there, and with easy_profiler, you'll be able to see which one
runs when! I just want to point out that for the sake of lisibility of this blogpost, I've set my NR_OF_WORKERS to 4 instead of the 14 my machine is capable of as I've advised before.

<h2>
  SimpleAsync()
</h2>

Since we know that approximating PI using the Monte Carlo method is an embarassingly parallel problem, let's try to make a version of our above function that splits
the workload of approximating PI into as many separate threads to effectively use the whole CPU for the task rather than a single core.
To do this, we'll start by using the simplest multithreading facility provided to us by the C++ standard: the [std::async](https://en.cppreference.com/w/cpp/thread/async).
Those familiar with Unity Engine's C# coroutines will probably find the concept quite similar: std::async's allow you to define some arbitrary bit of code to execute and then launch it
asynchronously, then retrieve the result of the execution when it has finished executing if necessary using a [std::future](https://en.cppreference.com/w/cpp/thread/future).

While std::async's are easy to use, it's important to remember that they are <strong>NOT guaranteed</strong> to execute the task on a separate core, that decision is left up to the operating system.
As such, the std::async are great when you just need to kick off some asynchronous task and don't really care about the details of where and how that task is running.
A typical use of this facility in video-games would for instance be the loading of assets used in a level while the game is displaying a loading screen for the player:
we wouldn't want the program to just freeze for ~10 seconds, making the player think that the game has crashed when in reality it's just loading a large amount of data from
the disk. Let's try to use them, shall we? 

<img align="center" width="630" height="114" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/simpleAsync0.PNG">

For those not familiar with this kind of notation, it's called a [lambda expression](https://en.cppreference.com/w/cpp/language/lambda). If you're not aware of it, think of it as a
local function (which it isn't actually by the way) where we'll be defining what one of our worker threads should be doing.
We'll be capturing <em>e</em> and <em>d</em> [by reference](https://riptutorial.com/cplusplus/example/1951/capture-by-reference) to get it inside the lambda's scope: it's an argument
that should be common to all the worker subroutines. Since <em>iterations</em> and <em>nrOfWorkers</em> are just 64-bit wide unsigned integers, we'll just pass them as regular arguments
instead, by value. The subroutine should return the number of points inside the unit circle for the subset of the total workload it has processed.

All of this is already written for you, your task is to simply re-implement the PI approximating algorithm, but this time only process 1/NR_OF_WORKERS'th of the total workload.
Spoilers, this is how you do it:
<img align="center" width="442" height="297" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/simpleAsync1.PNG">

The only thing we've changed is that instead of processing the whole <em>iterations</em> number of iterations, it's only processing 1/NR_OF_WORKERS'th of it. Since <em>e</em> is
captured by reference, the numbers generated for each subroutine <strong>should</strong> be different (but if you're aware of memory sharing concerns, you can already see why this
approach is problematic, but that's for the next blogpost).

Now that we've defined what our worker threads should be doing, let's kick them off so that they actually execute the code we've written. It's important to note that if you're launching
an std::async task with the std::launch::async argument, the destructor of the returned std::future becomes a blocking
operation, meaning that the main thread will not continue execution until the asynchronous task is done which would turn our hopefully parallel operation into a sequential
one, so it's important to store the std::futures aside until we need to retireve the results and not just kick off an std::async and discarding the std::future (which we need in our case
anyways since our subroutine returns a value):
<img align="center" width="757" height="110" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/simpleAsync2.PNG">

Now that we've kicked these tasks off, let's wait for them to finish executing and retrieve the results and use them to approximate PI, just as before. Note that the [std::future.get()](https://en.cppreference.com/w/cpp/thread/future/get)
is a blocking operation that only returns once the subroutine has computed a valid result.
<img align="center" width="665" height="178" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/simpleAsync3.PNG">

Good, let's launch our program and see how it performs... uh oh:
<img align="center" width="665" height="178" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusApproximatingPi/simpleAsync3.PNG">


<strong>TODO: image of a well behaved cat besides some yarn</strong>