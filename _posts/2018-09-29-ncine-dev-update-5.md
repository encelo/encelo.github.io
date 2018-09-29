---
layout: post
title: nCine Dev Update 5
subtitle: Updates from July to September 2018
tags: [nCine]
---

During those months two very important features appeared in the nCine.

The first one is the integration of [Lua](https://www.lua.org/) for scripting, a language which is very easy to integrate and runs very fast.
With Lua the user can quickly prototype ideas or actually write the entire game with just scripts, in a way similar to other engines.

There is also a second way of interacting with the language, as the engine has now a bunch of useful methods to exchange data with it.
You can load data from Lua tables to use a data-driven approach, or you can script the behavior of specific game objects.
Scripts can also be reloaded while the application is running, for a quicker iteration time and easier tuning of the game.

The second feature is the integration of [ImGui](https://github.com/ocornut/imgui/), a widely used immediate mode GUI that allows the engine to sports more meaningful and detailed performance data in a brand new debug overlay.
The old on-screen HUD has been rewritten and plenty new statistical plots have been added, including a new section dedicated to Lua showing things
like the amount of memory used or the number of operations per second.
While integrating ImGui, the rendering system was enhanced in order to accomodate the *scissor test* and more flexible options for indexed drawing and vertex formats.

<iframe width="640" height="360" src="https://www.youtube.com/embed/PQRnxeBpo-c" frameborder="0" allowfullscreen></iframe>

I have also worked for a month on *ncParticleEditor*, an editor for particle systems that use both new features: Lua for loading and saving and ImGui for the interface.
As the editor is really just an nCine application, it can run with no problems on Windows, Linux, macOS and of course Android!
The particle system was overhauled to support additional features like new particle affectors, a new particle random initializer structure and grayscale sprite rendering.

I have to thank my friend [Helba](https://www.linkedin.com/in/marcolisci/) for his infinite patience while testing the program and for allowing me to show some of his projects in the video.
To help with testing I have integrated [CrashRpt](http://crashrpt.sourceforge.net/) in my debug Windows builds to ease the creation of [Minidump](https://docs.microsoft.com/en-us/windows/desktop/debug/minidump-files) files.

<iframe width="640" height="360" src="https://www.youtube.com/embed/RLNI5NMCJ1E" frameborder="0" allowfullscreen></iframe>

I hope you have enjoyed both the videos and the new features. :wink:
