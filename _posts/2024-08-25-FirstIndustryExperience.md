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

While most of the systems I've had to work on were fairly straight forward once I poked around in them for a while, some of them are bordering on dark magic to this day. One of these major systems was the game's save system.
I had arrived on the project right when Simon was implementing a saving mechanic into the game, and that was namely one of the reasons he needed a programmer. Simon had followed the tutorials of a youtuber called Reid (I unfortunately can't
find the exact name anymore) and implemented a C++ save system in the project... but he didn't understand C++. So of course, once things were inevitabely broken, he was unable to fix the code to make it work for the project.
Instead, for the parts that were broken, Simon ended up making a second save system, on the Blueprint side of the engine, just for those last bits of data that needed to be saved.
That is when I've arrived on the project, and after a few months, I needed to stabilize these systems, as well as add some more functionalities to it. Namely, along the player's stats and the world's states, we needed to be able to save
the player's progress and upgrades and the player start that was to be used to respawn.
Just like every other system in the project, it's usage was chaotic and spread across 50 or so classes. This task turned out to be very much a "Weko-ism" as I will call these from now on: it worked 75% of the time, it was a pain in the bum
to extend it, and following any changes to it, a lot of playtesting would be needed to ensure everything is STILL working as intended. It was either that, or a month or so of code rewrite, which at such small scale of a team was not
justifiable.
Unlike Blueprint systems, errors on the C++ side of an unreal project usually end up in crashes, and it took a lot of trial and error, and well as guesswork to iron out all of the crashes and save corruptions linked to the C++ side of the
save system. And so, after a few initial weeks of work on it, followed by a lot more debugging for the remainder of the project, this system was stable enough to be released with.

Another such "Weko-ism" of the project has been the inventory system. The game, being an RPG, obviously has a lot of items. Some of them you can equip, some you cannot, some you can give to NPC's, other you keep for the whole game. Therefore,
one of the main systems in the game is the player's inventory. The player can move or equip items around by drag and dropping things with a mouse, double clicking, or by using gamepad buttons.
It goes without saying that this mechanic is spread across ~20 or so classes, Weko_BP being one of them. While making the save system work with the inventory was a challenge in of itself, the real trouble maker of the system turned out to be
Unreal Engine itself.
The engine has two ways to handle input as far as I can tell: the Unreal Engine mode, referred to as "Game" input mode, and Slate's input handling. Slate is a third party UI framework upon which Unreal's UMG interface is built.
It's hard for me to say whose fault that is, but while Unreal provides a nice an unified way of managing player input, with priorities, generalization of input hardware and so forth, this is not the case with UMG. In Unreal, input seems to be
processed in two very different manners, and for our use-case, in two mutually exclusive manners. While Unreal does have an "Game and UI" input mode, we weren't able to use it with our player pawn, and so the player's input had to be very
carefully managed in a game of input responsability passing, lest the game would end up in a state where user input is being ignored by both handlers of inputs (game side and UI side).
Add to that the fact that the passage of time is handeled differently by UMG and Unreal's Actors, the idea of implementing the ability to pause into the game has long scared us, until at last, following player feedback, we managed to hack
something together.
The most frustrating, and bug-prone part of the inventory's code was the fact that the game supports both a keyboard and mouse control scheme and gamepad controller schemes. On the Actor side of things, this distinction is very minor, Unreal
making a great generalization of inputs. On UMG's side however, things are very different. Unlike for regular inputs, inputs for UMG had to be hard-coded into the ~20 or so classes, and so, any iteration of the control scheme for the game
would often result in needing to manually change those bindlings, which was both time consuming and error-prone.
And lastly, with the gamepad control schemes especially, a lot of care needed be given to "focus" passing. In UMG, only the currently "focused" widget receives inputs from the player, unlike with Unreal's Actors, and so if the focus was lost,
one way or the other, with a gamepad, that would essentially imply a softlock of the game. And of course, UMG does not provide a function to retrieve the currently focused widget to help the matters.
However, after much headache and a lot of lessons learnt for the next project, that too, was stabilized. Not perfect by any stretch of the imagination, in fact you can still percieve some oddities with the game's UI today if you pay good attention to it,
but these oddities aside, the player is able to interact with the inventory and all of the related UI systems without bothering bugs.

And creating my own.

What happens when you give an inexperienced, fresh-out-of-school programmer the responsability of a new, major feature in a game? That was the situation I had found myself in during the summer of 2023: the game finally was able to justify
implementing bosses in the gameplay! Initially, 6 bosses were planned (just a small number for a small team, just small enough to handle while also developing the rest of the 6+ hours of an RPG's gameplay), and so, the good programming student
that I was, I immediately saw this as an opportunity to finally implement a nice, well designed, reusable and expandable system, the way a true programmer would! At the time, we were also reworking the code of enemies of the game, and so,
I saw this as an opportunity to harmonize things, take what already exists on the MasterEnemy_BP side of things, generalize it and build a system able to be used for bosses and regular enemies alike.
And, it went exactly as you would expect. Luckily, with the time pressure and basic self awareness, I did not spend weeks figuring out the best design for the task at hand, and I say luckily, because as both the bosses and enemies would be iterated on,
it became very clear that there was no way I would have been able to account for all of the requirements that such a system would entail with the little experience I had.
Instead, the task ended up being an initial 2 months of development followed by regular tweaking for the rest of the time until release.
First, we started with the Golem Boss: a big stone giant that is invulnerable to any kind of attacks until he rushes into spiked pillars and stuns himself. Just like enemies, he is able to attack in melee... and that's where the similarities with
the main enemy class ends. Seeing the difficulty of abstracting two kinds of enemies that have very different behaviours and characteristics made me quickly realize that it was best to handle bosses and regular enemies with two separate bits of code.
That did not stop me from trying to regroup most of the functionalities of all bosses into a single base boss class however. It took a lot of overriding, expanding and conditional execution, but the game does have a base boss class that is responsible for
a good chunk of the logic of the bosses... but looking back at it, that was not a good idea.
For regular enemies, ones you will be seeing constantly, in many different variations, it's great to have a base class with Actor Components used to define some specifics, but for bosses, a common base class is a hinderance.
Such rigidity restraints creativity and most of the time, due to the very varied nature of game boss design, you just end up making hacks upon hacks to make specific mechanics work, all the while taking care not to break the general logic
that is being used by all the other bosses. In the end, the game only has 4 bosses (which is still a lot more than I would have expected), but I recon that 6 bosses would have genuinely been possible if we had taken the decision to
re-implement each boss from scratch rather than ensuring that the most amount of code was able to be re-used across bosses.

This is a good example of one of the lesson I've learnt in the making of this game: "better is the enemy of good". At school, we are understandibly taught good programming practices and are taught the basics of good software design, but in reality,
in small teams anyways, the reality is that it's very difficult to justify "proper code" over "working code". I'm sure the situation is very different with a bigger team of course.

As an example of this, consider the following mechanic from our game: the boss jumps in the middle of the room and prepares to fire a big, one-shot-kill laser that sweeps the whole room, but conviniently also drops down two stalagtites at
either side of him that allow the player to survive the attack if they hide behind them.
One problem that very quickly became obvious was that if the location of those two stalagtites was left up to random chance, the player would often end up being hit by the falling stalagtite, and then swiftly killed by the laser as
they get up. To fix that, it would be best to ensure that the two stalagtites consistently spawn on either side of the boss on a line that is perpenducular to that formed by the position of the boss and the player: this way, the player doesn't
get squished by the thing that is supposed to protect him and the two stalagtites are always within a reasonable distance from the player to ensure that they can reach them on time.
Now what would be the proper way of computing the location of the two stalagtites? Surely you'd need to use trigonometry and geometry to find these locations, take into account the rotation of the boss pawn, the direction of the player, and so on and
so forth.
Or... how about we just generate one random position on a circle around the boss and check that it's not too close to the player? And if it is, eh, just generate another one. Then use that position to spawn one stalagtite and compute the other
position for the second stalagtite on the other side of the boss, easy! What's that? The player still gets hit by the second stalagtite because the first one has spawned behind the boss, relative to the player? Well, just check that the distance
of the first stalagtite is neither too large nor too small, and tada! You get stalagtites that spawn on either side of the boss consistently and that do not crush the player when they appear.
Now, when I've described this solution to a junior classmate of mine, he very understandibly facepalmed upon hearing that nonsense.
But... that is currently how the stalagtites of Cinos, the third boss of the game spawn. Why? Because it works consistently and unlike with the "proper" approach to the problem, I did not have to sketch out the math problem, which is a good thing
since I'm not that good at it (there's a reason I had trouble with getting hired by big studios!). Mind you, that does not at all mean that I couldn't figure out the proper way for solving that problem, just that it would have taken me more time than this
one. But for what, to feel satisfied? No, that time was much better spent debugging the rest of the game and implementing the rest of the mechanics. I am the only person who has had to touch that particular bit of code and it wasn't a feature
that has been re-used anywhere else, nor was this solution a weight on the performance of the game, it is just unsightly. But that, I guess is how you make a small Zelda-like adventure game with only 5 people that has a playtime of over 6 hours.

A pawn in the greater scheme of things.

With this first industry experience, it was the first time that I occupied an important position in a project without having a major say in it. Up until then, for group projects, I had usually taken a leadership role, but arriving in this project,
half-way in, I of course assumed a more passive role. Not that my input wasn't valued, quite the contrary, I've never felt like my suggestions or ideas were discarded, unjustly, but having had some limited experience in making sure a project arrives
to it's term, it was an nice change to not have to worry about anything other than programming.
This has really made me appreciate the invisible work that Simon and Robin have done during the time I was able to just focus on programming. Up until now, you might think that I've been throwing Simon under the bus, critisizing his old code but
that is very far from it. Seeing what Simon was able to accomplish alone, programming wise, is quite impressive considering the scope of the project. Of course someone without a formal software design education would make mistakes, and for all the
grief I've given him here, I absolutely cannot guarantee that I would be able to make "proper" project, programmng-wise, I am a junior with only a little theoretical knowledge in software design.
Where I really have to give credit to Simon is pretty much everywhere else.