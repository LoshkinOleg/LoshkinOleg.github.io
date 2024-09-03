---
title: "Wéko: My First Experience in the Industry"
date: 2024-08-25
layout: post
categories: blogpost
permalink: /:categories/:title
---

<br>
<img align="center" width="674" height="446" src="{{site.assets_dir}}/Blogposts/MultithreadingInCplusplusProducingAndConsumingPi/pie-eating-contest.jpg">
<em>[Promotional picture for a pie eating contest](http://www.lauradays.org/pie-eating-contest.html)</em>
<br>
<br>

This blogpost is a retrospect on my time at Siro Games sàrl during the development of Wéko: The Mask Gatherer. So, a few introductions are in order. Firstly, Siro Games sàrl is a Geneva-based, Swiss indie game development company.

How it all began.

I have arrived at the company thanks to Lisa Manolache[https://www.linkedin.com/in/lisa-manolache-89a7b1197/], a talented Game Art classmate I've had the pleasure of collaborating with during our last year at school.
At the time, I had been looking for a job for about 5 months, without much success: in such a competitive industry, it isn't easy for juniors to find any position. Luckily, Lisa had heard from Simon himself, one of the two
founders of Siro Games, that they have been looking for "a dev" for the game. More precisely, it turns out that they were looking for a gameplay programming intern, and my profile fit the bill!
So, without much hope, I had sent Simon my candidature, as I had already done to other companies, but this time, things went quickly and I had an interview. Unlike all the other interviews I've had before, this one
did not focus on C++ semantics, abstract challenges consisting in reinventing Microsoft Excel or in listing the names of some arcane computer science algorithms, no, for this one, the interviewer had asked me something shocking:
"Would you be able to help us with this particular project, and how?".

Wéko: The Mask Gatherer.

The project in question, was, of course, Wéko: The Mask Gatherer. Wéko was a project that was in the making for one and a half to two and a half years, depending on where you would consider it's start to be.
The genesis of it, according to Simon, was surrounded in that from which many such good ideas stem: beer!
Simon tells me of an evening, when he and Robin, the other founder of Siro Games, were having a beer and discussing their shared interest in video games. Simon, a data analyst in the banking industry, had some good experience in
writing scripts and wrangling with Microsoft Excel. And Robin, is a very talented jewler with great 3D sculpting skills. With some alcohol thrown into the mix, it was only natural that the two decided that it would be a 
good idea to make a fully 3D, action adventure game remenicent of the recent Zelda and Dark Souls games, with no prior experience, funding, connections or marketing of any sort, all in a canton that barely acknowedges the
existence of video games and in one of the most expensive countries in the world!

And so, make a video game they did. Over the first two years, development went understandibly slowly, as Simon and Robin would learn the basics of Unreal Engine 4.27, the engine the duo had chosen for the game.
Over time, Simon would reduce his hours working at the bank to 50%, and would use the remaining 150% of his free time (who needs sleep anyways) to work on the project. Eventually, our crazy duo would be joined by
the very talented Quentin, the music composer of Wéko's OST and Rimayé, the VFX wizard responsible for all the flashy and cool visual effects you can find in the game. There were many others of course such as Liam, the cool
theater kid who inexplicably figured out how to tweak and create animations for us, and the various people that have helped us with translations later on, among others. If you want the full list of people that have worked
on this game, I encourage you to check out our game's credits (once you've bought it, mouhehehe)!

Humble beginnings.

By the time I've had that interview with Simon, most of the core mechanics of the game were already there... kind-of. While I did create quite a few mechanics and features from scratch, the majority of my time on the
project was spent stabilizing and on rare occasions refactoring the existing code. Simon has had experience with scripting, as I've mentionned, but he lacked the rigor necessary to create the sizable reliable game systems
such that were needed by this project. When I arrived, while most of the mechanics were present, they would fail to function properly 1/3 of the time, or would consistently fail as soon as they interacted with each other,
such as carrying an object and falling off a ledge.
Things were working "well enough" in isolation, but would become completely unpredictable or outright game breaking during the course of normal, multi-hour spanning play.

And so, when I arrived, a lot of time was spent organizing the existing code, and oh boy, that was NOT an easy task. As a reminder, the project was now a year and a half worth of action adventure game code cobbled together just
enough to work most of the time with little to no consideration given to basic good coding practices such as separation of concern or encapsulation (worse yet, encapsulation that does not actually encapsulate anything). If I had
needed to get the whole codebase in "well done" order, I would still be working on it today. With the given constraints, that was simply not doable nor sensible to do. No, what I needed was to stabilize this castle of cards just enough
for it to be able to withstand 95% of the things that the player might throw at it and the eyes of any programmer be damned!

Thus, my laborious task began. After a couple of game mechanics implementations that we didn't end up using, I got to work on Weko_BP, the player pawn's class, the monolith that regrouped at least 30% of the game's code.
After much despair, head scratching and harassing Simon, who had born this thing into existence, I had organised the player's code into a finite number of states, like "None", "Attacking", "OpeningDoor", "Dead" and so forth.
With this, tracking down bugs slowly became easier as we were now able to track down the sequences of events that would lead the player to end up with member variables set to incoherent values for the current context of the game... most of the time.
Again, in all I've done, I had to make careful consideration of what needed to be refactored and what was too fragile and intertwined with everything else to hope to rework within a reasonable amount of time.
Where I couldn't rewrite things, either because that would take too much time or because I simply couldn't understand all the mirriad of effects changing a single value would cascade into, I've done my best to add failsafes,
to write sanity checks and if all else failed, just comment everything for the next time my bug hunt would lead me down to that particular piece of the code.

After a few months of this, with Weko_BP as well as other, similarly convoluted areas of the codebase, I was finally starting to get the big picture of what was happening in the game at any given moment... for most of the systems, anyways.

Facing the monsters.

