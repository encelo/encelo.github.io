---
layout: post
title: nCine Dev Update 11
subtitle: Updates from June to the first half of July 2019
tags: [nCine]
---

Exciting news for this development update: a new supported platform! :champagne:

### Emscripten

I remember playing with the idea of porting the nCine to [Emscripten](https://emscripten.org/) years ago.
After all I had every requirement in place: I used OpenGL ES for Android, GLFW and SDL2 as input backends, OpenAL and Vorbis for audio, libpng for images and already supported a POSIX API.

Unfortunately there was always a showstopper preventing me to achieve some progress and gain more motivation.
I remember managing to compile unit tests and run them from the console with Node.js but that was it, no apptest was ever working or sometimes even compiling.

This time was different, I studied the documentation in detail, put a lot more energy and dedication but here we are, it works!

Of course there were a lot of little issues that ended up eating plenty of time. :weary:

#### Rendering

I encountered the first issue when I tried running apptests on Windows, the browser just froze.
Further investigations led me to the culprit: shader compilation was super slow when using [ANGLE](http://angleproject.org).

I did some research on the internet and I stumbled upon the [Asynchronous Shader Compilation](http://toji.github.io/shader-perf/) demo.
The article explains a way to give browsers a chance to compile shaders asynchronously while performing other loading related tasks.

I followed the advices: compile and link shaders but defer any error checking or introspection to the first use.
It didn't fix the issue at all but made loading time faster on every platform. :sweat_smile:

Just have a look at the following traces and compare how much `initCommon()` takes with immediate ("_This trace_") versus deferred ("_External trace_") queries.

![Tracy_deferShaderQueries](/images/Tracy_deferShaderQueries.png "Tracy - deferShaderQueries")

I then consulted a friend about ANGLE and he concluded that the long arrays I used for collecting batching instances information were the cause, and that the Direct3D HLSL compiler is well-known for having a hard time with them.
After redeclaring those as single element arrays the compilation speed was reasonable again.

If the shader declares a single element array then it should only be able to access that element, even if you bind a bigger buffer, right? According to my tests that's not true and every GPU on every OS out there will perform an out of bounds access and render every instance.
Well, every combination except Catalyst drivers on Windows, therefore restricting these shaders modifications to Emscripten only.

But batching was broken no matter what, with or without single element arrays there was always some instance not being drawn.
I suspected an issue with _Uniform Buffer Objects_ alignment as `apptest_camera` with culling enabled rendered flickering sprites, very similarly to what happens when alignment is incorrect.
I tried for days with different alignments and buffer sizes but I didn't have any luck and I decided to disable the feature on Emscripten for the time being. :cry:

When I was lucky rendering fixes came from extending Android OpenGL ES specific `#define` conditions to include Emscripten, like going from `#ifndef __ANDROID__` to `#if !defined(__ANDROID__) && !defined(__EMSCRIPTEN__)`.

Buffer mapping might have been another big issue as it is not supported on WebGL 2. Fortunately it gets emulated when you pass `-s FULL_ES3=1` to *emcc* and it's an optional engine feature that can be disabled.
This way I got the code to compile with no modifications while avoiding a possibly slower emulated path.

#### Input backends

Rendering was not the only area in need for a change. On SDL2 auto-suspension on focus lost just hangs the application indefinitely. That is caused by `SDL_WaitEvent()` and it appears the only fix is not to call this function.

Emscripten also doesn't seem to play well with GLFW windows at the moment, it needs a little help in the form of a `glfwWindowHint(GLFW_FOCUSED, 1)` when you open one.
I also had to exclude some GLFW 3.3 code I added recently, but nothing that couldn't be fixed by a `#if GLFW_VERSION_MAJOR == 3 && GLFW_VERSION_MINOR >= 3`. :wink:

At least joystick mapping worked without much hassle, I just had to return the special "`default`" GUID for every connected joystick in order to match the Emscripten entry in the SDL2 game controller database.

#### Additional changes

After guarding [death tests](https://github.com/google/googletest/blob/master/googletest/docs/advanced.md#death-tests) with `#ifndef __EMSCRIPTEN__`, unit tests were also good to go. Thanks to the Emscripten CMake platform script, running `ctest` invokes `node` as the `CMAKE_CROSSCOMPILING_EMULATOR` allowing unit tests to run from the console. :muscle:

Apptests were also mostly working, with the exception of `apptest_simdbench`. I had to refactor it in order to use a lot less memory and allocate it from the heap.

As you might have already [read](https://emscripten.org/docs/porting/files/file_systems_overview.html) on Emscripten website, accessing the file system is different too.
The C API is emulated, and that's very useful, but you still have to provide data to your web application in a special way.

I decided to go with *preloading* instead of *embedding* to always have a separation between data and code.
For apptests I directly call the [file packager tool](https://emscripten.org/docs/porting/files/packaging_files.html#packaging-using-the-file-packager-tool) within CMake, I generate the data once and share the file between them.

The `nCine-libraries` project also gained Emscripten support and it can now compile Lua and WebP for the new platform.

### OpenAL fixes

While working on the port I had some people try my [web tests](https://ncine.github.io/web-tests/) and a tester reported a strange issue with music and sound effects in `apptest_audio`.
In the beginning I thought it was something related with the new platform but the issue could be found on all of them. :scream:

The OpenAL implementation was probably one of the oldest pieces of code, it had not been touched in a very long time and it showed. :sweat_smile:

First of all the source id recycling was pretty much wrong and paused or stopped players never relinquished their ids. Also, they could never transition from a paused to a stopped state. :weary:

Even if the number of OpenAL sources is fixed and there is only one source per active player and vice versa, I was managing currently playing players with a list.
The optimization was as easy as to swap `nctl::List<IAudioPlayer *> players_` with `nctl::StaticArray<IAudioPlayer *, MaxSources> players_` and change some accessing code. :wink:

While I was there I took some time to sprinkle a couple of `alGetError()` calls around source and buffer generations, just as suggested by the [OpenAL Programmer's Guide](https://www.openal.org/documentation/OpenAL_Programmers_Guide.pdf).

I have also added querying methods for buffers, streams and players that allowed for the creation of a new section in the ImGui debug overlay, one showing information about active audio players.

![ImGui_AudioPlayers](/images/ImGui_AudioPlayers.png "ImGui debug overlay - AudioPlayers")

### Monitor video modes

Last but not least, there is now a way for games to query and change monitor video modes on PC.
The implementation uses functions like `glfwGetVideoMode()` and `glfwSetWindowMonitor()` on GLFW and `SDL_GetDisplayMode()` and `SDL_SetWindowDisplayMode()` on SDL.

The ImGui debug overlay has been updated to take advantage of this new feature. The user can select a full screen video mode from a drop-down list of supported resolutions and refresh rates combinations.

![ImGui_WindowSettings](/images/ImGui_WindowSettings.png "ImGui debug overlay - WindowSettings")

That's all for now and I hope you are excited about porting your nCine projects to the web. :spider_web:
