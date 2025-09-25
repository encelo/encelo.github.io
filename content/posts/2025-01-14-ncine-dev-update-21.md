---
date: "2025-01-14"
title: nCine Dev Update 21
description: Updates from December 2023 to December 2024
categories: [ Dev Update ]
tags: [ nCine, OpenAL, AppTests, Multi-Threading, Lua ]
series: [ Dev Update ]
series_order: 21
---

In this article, we’ll go over the progress of the nCine throughout 2024.

## OpenAL EFX

The biggest change this year has been support for the OpenAL EFX extension. You can now apply effects to any audio player, like reverb, echo, flanger, and more, and use low and high-pass filters.

This was also a chance to improve the OpenAL code by adding new features and fixing some old bugs.

For example, you can now adjust the OpenAL context attributes to change the number of audio sources or adjust the frequency. Sources now support more properties, like velocity, direction, and cone angles. With the EFX extension, even more properties become available.

There’s now a pool of OpenAL sources that are assigned to players as needed. This lets you have more players than hardware sources, as only those actively playing need a source. You can also lock sources to frequently used players to avoid the system from reassigning properties. And if there’s an OGG audio dropout (like an `OV_HOLE` error), decoding continues without stopping.

The new effects and filters API is also available in Lua, and the `apptest_audio` application has been updated to test both it and the source pool.

## Multi-threaded job system

Another major feature in progress is a multi-threaded job system, though it’s not finished yet.

I’ve been thinking about addressing CPU bottlenecks in the nCine by implementing a data-oriented ECS with a multi-threaded job system. I started working on the [molecular matters](https://blog.molecular-matters.com/2015/08/24/job-system-2-0-lock-free-work-stealing-part-1-basics/) job system, which includes a lock-free work-stealing queue, parent-child jobs, continuations, and parallel for loops. Their articles provide a simple implementation of a solid system that seems to fit my needs well.

While making the logger thread-safe using a queue of messages, I paused work on the job system to focus on something else.

## Lua development improvements

While working on the job system, I came across the Lua Language Server and decided to add support for it. Improving IDE support for the nCine with Lua has always been on my mind, and LuaLS was the right tool for the job.

It took a few weeks to collect all the Lua API functions into [LuaCATS](https://github.com/nCine/nCine-LuaCATS) definition files, but it was worth it.

{{< youtube id="vyXqnrW5_5Y" loading="lazy" title="The new Lua development workflow with the nCine" >}}

As shown in the video, using the Lua extension for Visual Studio Code now gives you autocomplete, type checking, and inline API documentation. This makes the nCine experience similar to using frameworks like LÖVE 2D or Solar2D. The LuaCATS work also made it easy to create a new [LDoc documentation](https://ncine.github.io/docs/lua_master/) that’s available online.

Reviewing the Lua API also allowed me to fix missing function exports, rename inconsistent methods, and write documentation that filled in missing Doxygen comments for the C++ API.

![Lua Language Server](/images/Lua_Language_Server.png "Lua Language Server")

### Improvements to `apptest_lua`

The `apptest_lua` application, which becomes the `ncinelua` executable in the Lua Distribution version, is the tool users use to write their games in Lua.

It already supported features like script hot-reloading and basic on-screen error display, but not all errors were shown, and users often had to rely on the console log.

Now, errors from application callbacks like `on_init()` or `on_frame_start()` are displayed on-screen, just like in the console. I also added support for tab characters in the `TextNode` class, so Lua call stacks are properly indented. :smile:

The executable can now be run with `-h`/`--help` and `-v`/`--version` parameters, making it more like a standard command-line tool. To support this, I added a way to quit the framework from the `onPreInit()` callback, so no initialization happens, and no window briefly appears. You can also disable logging from this callback to completely silence the framework. 🤐

### Minor changes

- Joystick axes mapped as buttons are now supported
- Added new constructors to the `MemoryFile` class to take ownership of externally allocated buffers
- Fixed a typo in a preprocessor conditional that prevented joystick hats from working in GLFW
- Moved the `Particle` class header to public headers to fix a forward declaration warning in `ParticleSystem`
- macOS builds now use two GitHub Actions runners, with one compiling nCine natively for Apple Silicon
