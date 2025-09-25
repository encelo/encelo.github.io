---
date: "2021-06-21"
title: Ten years of nCine
description: A retrospective ten years after the first commit
tags: [ nCine ]
---

A bit more than ten years have passed since that [first commit](https://github.com/nCine/nCine/commit/6bf318de68ed5c453eaacd867c8e83c853f64edc).

The presence of a `.hgignore` file reveals that I was using Mercurial at the time, an easier transition to DCVS for someone like me used to Subversion.

Some things were already there and stayed the same until now: like the Doxygen comments or the CMake building process (my first CMake real project after years of SCons).
Many other things have changed instead, as can be expected from a project with such a long life span.

But what made me persevere for a decade on the same project?

![nCine Logotype](/images/nCine_logotype.png "nCine logotype")

## The early years

I have been publishing open-source software for more than 20 years. :floppy_disk:

The very first was [MiniStat](http://aminet.net/package/dev/src/MiniStat), published on Aminet in 2000, a very simple program to perform some statistical calculations that I wrote to practice what I learnt after reading my first C course.
At that point, I had been an Amiga user for ten years and even if I was extremely fascinated by games and the demo scene, I was also quite attracted to system programming.

So when I became a Linux user shortly after, I oddly decided to work on a CGI script. I had recently finished reading the [GaPiL](https://gapil.gnulinux.it/) book about Linux programming and those were the days of slow always-on ADSL connections and home servers.
That's how [Sonda](http://sonda.sourceforge.net/) (2002-2003) was born, a CGI script for user polls written in C and hosted on a friend's home server. :smile: The project was my first contact with SourceForge, with licenses, with CVS, and later with SVN.

Then Doom 3 came out and my interest switched back to games and shiny graphics. First I joined my good friend Vivaladav to work on [Mars, Land of No Mercy](https://sourceforge.net/projects/mars/), a turn-based strategy game with isometric graphics, and then I started experimenting with [OpenGL demos](https://www.autistici.org/encelo/prog_gldemos.php).

With [GL O.B.S.](http://globs.sourceforge.net/) (2006-2007), my next big project, the focus stayed more or less the same: graphics, performance, and tools.
Globs was a benchmarking solution based on a PyGTK interface, a PHP script to gather user results, and my OpenGL demos as individual benchmarks.

The tool was even used briefly by [Phoronix](https://www.phoronix.com/scan.php?page=article&item=599&num=7) long before OpenBenchmarking.org was a reality. :scream:

Later I had some time to deviate a bit from graphics with [PacStats](https://github.com/encelo/pacstats) (2007-2010): a little PyGTK project that extracted information from the Arch Linux pacman log file, stored it in an SQLite database, and visualized it with Matplotlib.

Finally, in July 2009, I graduated with a thesis on computer graphics and I was ready to join the game industry! :muscle:

## The beginning

I looked for a job for a year, doing phone and on-site interviews in the UK with no luck. I remember I was considering going indie with a twin-stick shooter I was programming with XNA. :laughing:

But then the opportunity presented itself and I started working in a small indie studio in Italy.

While I was daydreaming about AAA games, rendering, and engine programming on next-gen consoles, the reality was a lot different: I was stuck with GUI programming with no escape routes.

But that is the precise moment the nCine was born: at work, I was coding boring user interfaces but at home, I could be whoever I want, and I wanted to be an engine and graphics programmer! :gear:

My first two working machines were two very small and slow netbooks: a [Medion E1222](http://encelo.netsons.org/old/my-computers/atom-2/) first and a [Lenovo IdeaPad S205](http://encelo.netsons.org/old/my-computers/gluon/) a bit later. :tired_face:
But they had all I needed: Arch Linux, Qt Creator, GCC, CMake, and Doxygen. Those were my main tools then as they are today.

When I planned my work I deliberately decided to start with simple rendering: just 2D sprites and no shaders. My main focus at the time was the engine architecture, the data structures, the sound system, the abstractions... all things that I neglected before.

Of course, I carried my obsession with "perfect" commits to this new project. My policy dictates that I rebase a commit if I later discover an issue as if every commit was a micro release itself.
This constant rebasing is the reason why there are very few commits and also makes a collaborator's life harder. Fortunately, I have none. :sweat_smile:

## The consolidation

Over the years I changed jobs several times to come closer to low-level graphics and engine programming. This didn't stop me from working on the project.

My two jobs in the UK were centered around mobile and Android development, and Android became an important platform for me.

At the time it represented the closest thing to a console accessible by indie developers: those were the days of OUYA, GameStick, Gamepop, MOJO, and Android TV set-top boxes like the Shield or the Fire TV.

When I began working on the nCine I knew I wanted to release it as open-source one day, but I believed it was too rough and I always delayed the date.

Both my employers in the UK allowed me to release the source code whenever I wanted so I thought it was not going to be a problem with my new company in Sweden. Unfortunately, I quickly learnt that their legal team was very strict on this point: I could never release the source while working with them.

So I continued to work behind closed doors but I started to open up a bit to the world. I wrote development updates on this blog, distributed binaries to some friend developers, and opened a [Discord](https://discord.gg/495ab6Y) server.

Around this time I began exploring the idea of creating a real game with the nCine. I put together a [list](https://docs.google.com/spreadsheets/d/1z1RV6w4HjPODFcao34h72Vw-9KxtQ9rv0ZM7folTUMU/edit) of 2D games made with custom engines by small indie teams to motivate me and keep the dream alive. :smile:

## The present day

Being unable to make my work public was one of the reasons I quit my job. I moved thousands of kilometers to the south, in Spain, and for two years straight I worked day and night on my projects alone.

I started by working on a new game: a turn-based isometric prototype code-named `ncIsometric` but after just a couple of months I got stuck with the utility AI code and put it on hold. :sweat_smile:

The second, more important effort, was keeping the promise with myself and release the engine as free and open-source software. The 30th of May 2019 was the big day: the nCine source code was released on GitHub with an MIT license! :godmode:

It was covered by both [Phoronix](https://www.phoronix.com/scan.php?page=news_item&px=nCine-Game-Engine) and [Game From Scratch](https://gamefromscratch.com/ncine-2d-open-source-game-engine/), and this initial press coverage that quickly raised the number of GitHub [stargazers](https://github.com/nCine/nCine/stargazers).
The stars showed the appreciation of the people and made me happy, but what I needed was battle-testing it on the ground: could others use it to create beautiful games?

This was a very important point because before even feeling the excitement of creating a game, I was longing for the excitement of creating a tool that someone else could use to create one.
I believe this weird indirect feeling defined me as an engine, graphics, and tools programmer: the satisfaction of making people happy and grateful for my support and efforts.

At this point, I created a template project to make it easier for potential users to develop games and a command-line tool to automate the download and compilation process.

Then the unexpected happened: Jugilus asked for help to support the nCine in [JugiMap](http://jugimap.com/), his 2D level and map editor. The tool lets you create levels, animations, collisions, and some logic, then a runtime library loads the data and uses the underlying engine to render everything.

JugiMap's runtime was a great way to stress-test the nCine and reveal a lot of bugs and limitations that, once addressed, made it a lot more capable and solid! :muscle:

Then it came the time for me to create my tool: [SpookyGhost](https://encelo.itch.io/spookyghost). By now you already know the joy I feel when a content creator uses one of my tools and I have wished to create one for pixel artists for a long time.
Both because I love pixel art and because I always wanted to create a game with this style.

I planned to sell it on Itch.io, get rich and start an indie company about game technology, tools, and game development. Naturally, nothing of that ever happened, I sold a handful of copies and I had to return being an employee in the game industry. :sweat_smile:

While a traditional job means less spare time, it also means more economic resources. Those resources might be invested in the project one day: they could translate into a marketing campaign or in a game jam with prizes. :trophy:

Speaking of economic resources, at the beginning of this year I was awarded the [Icculus MicroGrant 2020](https://icculus.org/microgrant/2020/), a small help but a great recognition for my ten years of hard work.

Besides, now that I don't need the money so bad I might as well make SpookyGhost an [open-source](https://encelo.itch.io/spookyghost/devlog/256745/spookyghost-is-now-free-and-open-source) software and hope more artists will notice it. :thinking:
And that's precisely what I did: I recently released it on GitHub under an MIT license hoping more users will try it.

## The future

After all those words, where are we today and where are we going tomorrow?

Ten years have passed but the user base is still very sparse and small. The majority of people I come in contact with will just test it a bit and disappear in the end.
I can't blame them, the project is not well documented or marketed and there are no real games made with it, so why should they risk and be the first?

For this reason, I often think I should maybe be the one creating something: a simple but polished game that shows the capabilities and the potential. It would also satisfy my ancestral dream of creating a game completely from scratch.
The main issue is that every time I start with a game project I end up adding a missing feature or fixing a bug in the engine. :smile:

That's our curse, engine programmers, always thinking at the foundations and never at the finished product.

A question I get sometimes is how I managed to be focused all this time on the same project. Well, the nCine initiative is a collection of different projects, I never get bored and I can always start a new one.

What keeps me going is the dream of seeing a successful indie game made with my technology. I work on my project a bit every day, with both small actionable tasks and a long-term roadmap, treating it as a real product but never missing out on the opportunity to have some fun. :wink:

I hope you enjoyed the journey so far and I hope to have the strength to keep going for at least another ten years. :muscle:
