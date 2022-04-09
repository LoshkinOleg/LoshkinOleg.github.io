# OpenGL Enums Cheat Sheet

## Some Context

As part of the 3rd bachelor year’s curriculum, students of the Games Programming section of the school are tasked with creating a video-game over the span of 7 months (end of september to end of april).
During the scholar year of 2021 to 2022, the school, along with the support of its students, has organised a collaboration between the Game Programming, Game Art and Sound Engineering students which resulted in the project that culminated in the creation of Voliday, a city builder wholly created using the video-games’ industry’s standard tools.

This culminated in 35 people having to all work in one way or another on the Unreal Engine project files, the engine picked for the making of the game. The team members were split into 4 teams.
The Game Artists Team, once the production of the assets was done using their own modelling and texturing software, used Perforce to integrate their assets into the game. They also shared assets among themselves using Google Drive.
The Sound Engineers Team, once the production of the assets was done using their own sound production software, used Wwise for sound design and for tweaking the sounds. The Wwise project was then integrated into the game using Wwise’s official Unreal Engine plugin. The Wwise project and the Unreal Engine project was hosted on a local Perforce server at school.
The Gameplay Programmers and the Tool Programmers both used Github to host a Git repository of the Unreal Project, I will go into why below.

While my official role was that of a Project Lead, due to personal circumstances of the Lead Programmer and the DevOps, I ended up supplementing their work. Namely, I ended up being responsible for the game’s code architecture and the deployment of Perforce, so I will be talking about these two responsibilities in this blogpost.

## Perforce

For those not familiar with it, Perforce Software is a Minneapolis based company that provides version control solutions and is the main provider of such software in the AAA gaming industry.
In the case of our project, as part of requirements issued by the Games Programming Head, we have used the Perforce Helix Core software to deploy, with the help of our teachers, a local on-premises Ubuntu server for hosting a Helix Core repository.

While the use of Perforce was initially planned to be the sole version control solution for the project, we quickly realised that due to various exceptional circumstances outside of our control involving the school’s local network we would not be able to use it right away. Therefore, the DevOps had taken the decision to start development of the game on Github to then switch to Perforce once the server became ready.
The server was ready only by mid-March and the educational licence for Perforce Helix granted in the beginning of February, at which point all programmers had gotten used to Git (with GitLFS for large files), managed by the team’s DevOps.
Switching from Github to Perforce was out of the question: the local only access to the Perforce server made it an impractical solution for many of the key programmers as commute for some of them would take over three hours daily and cost around 40.- CHF for the round-trip.

Therefore, only the Game Artists Team and the Sound Engineers Team have chosen to work on Perforce due to its more user-friendly interface.

This however created an awkward situation with two versions of the game being developed almost in parallel: assets would be added on the Perforce repository and the code would be done on the Github repository. This resulted in having to manually synchronise the two repositories regularly. While Perforce also offers Helix4Git, a tool meant to synchronise Perforce and Git repositories, it would not have worked when trying to synchronise a remotely hosted Git repository and a Perforce repository with local-only access.
This sub-optimal situation arose from the school’s networking issues outside of our control: a VPN service was originally made available to allow connection to the school’s local network remotely but said service was not functional starting in February.

If I had the benefit of this foresight, I would have probably instead invested in teaching the use of Git based tools to the Game Artists and Sound Engineers Teams instead or would have hosted our own Perforce server via a dedicated hosting service.

## Game's Code Architecture

While the responsibilities of a Lead Programmer were not initially my own, due to personal circumstances of my teammate, I’ve supplemented, with their consent, the role starting in December.
To be perfectly frank, the experience has been quite rocky to say the least: we had previously only one or two days of introduction to Unreal Engine and none of our programming teachers has had any considerable experience with this engine.
While one of our mentors, Fréderic Dubouchet, was able to demystify some of our concerns, we nonetheless were left nearly to ourselves in the learning of this new tool.
While general knowledge of game engines and game programming was certainly an asset when it came to learning to use this software, that did not keep us from making some blunders that are glaring in retrospect.

Firstly, thinking that using Blueprints would make development of the game easier in the long run, I had taken the decision in December, during the refactoring phase of the project, to make all code be written using this language.
This was a mistake as in addition to having to nearly re-learn how to program thanks to Blueprint’s visual approach to programming, the usage of Blueprints makes code conflict resolution extremely painful due to the fact that the code is no longer a mere textual file.
This resulted in a lot of re-writing of perfectly functional code due to the squashing of whole Blueprint files in case of conflict, forcing programmers to do the same exact work multiple times.

Making matters worse, I neglected to investigate the format that Blueprints take before refactoring the game’s code architecture, wrongfully and without a second thought assuming that evidently, any code, be it visual or not, would take the shape of a text file, thus making conflict resolution easy.
As a result, by 13th January 2022, our Programmers Team has re-started work on the project in a system that was not anywhere near enough modularised to allow efficient coding, the worst offender being the Building Blueprint class, that over the remaining months of work ended up accumulating about a fifth of the game’s core logic, making simultaneous work on it a nightmare.

Ever since the start of February when I realised my mistake, my advice to students junior to me has been consistent: when working in Unreal, set up your project as a C++ one and do not use Blueprints as those would be best left for the people it was designed for: Game Artists and Sound Designers. Using them as a programmer requires a hard adjustment in reasoning about code and is needlessly frustrating.
If you do however end up using Blueprints, it is crucial to well in advance anticipate as much future developments as possible and as much as possible, sometimes even to the point of absurdity, to encapsulate and modularise code. If I had to do this project over with Blueprints, I would have insisted on the creation of a dedicated code architecture tester role whose responsibility would have been to thoroughly identify the constraints and limitations of the designed code architecture. Using ActorComponents is absolutely key to working with Blueprints and manager type classes are in my experience the bane of Blueprint code.

## Conclusion

To sum things up, while the deployment of our version control solutions was sub-par, it has still offered our Game Artists and Sound Engineers Teams the opportunity to get acquainted with the ubiquitous version control software that is Perforce. While I regret that my programmer teammates did not have the opportunity to interact with it, I take comfort in the fact that the learning of a new production tool was not an additional challenge for them to face during the creation of this game. As for myself, this experience has allowed me to peer deeper into the mechanics of version control thanks to having to set up the software up myself on the server made available to us by our teachers.

As for my “vice-Lead Programmer” role in the project, I wish I could have done a better job of anticipating upcoming constraints that come with the use of Blueprints. My lacklustre performance in this role can be easily attributed to my accumulation of responsibilities that resulted in the diminishing of the time I had allocated for each individual role. If nothing else, this is a very good example of the more subtle effects that come from the “Hero Syndrome” phenomenon that I’ve detailed further in my blogpost concerning the human organisation aspect of the project (https://LoshkinOleg.github.io/Managing_Voliday_The_Human_Aspect).

