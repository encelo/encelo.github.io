---
date: "2019-05-14"
title: nCine Dev Update 9
description: Updates from April to the first half of May 2019
categories: [ Dev Update ]
tags: [ nCine, Lua, nCTL, Android, Rendering, Input, Tools ]
series: [ Dev Update ]
series_order: 9
---

It has been a month and a half of small but useful updates for the nCine.

### LibPNG

The PNG image loader has been modified to support more color types, by copying some code from the libpng [example](https://sourceforge.net/p/libpng/code/ci/master/tree/example.c). It means that any nCine game is now able to properly load PNG images with palette or with gray-alpha channels and to expand or strip bit depths that are different than the standard 8 bits.

### Lua fixes

I have also sucessfully converted the ncPong test game to a Lua script and fixed a lot of Lua bindings bugs in the making. :bug:
Things like missing methods or constants that were never exported to the [Lua API](https://ncine.github.io/lua_api.html).

There are also some Lua utils additions and it is now possible for games to set and retrieve fields directly or to set and retrieve globals.

### Headers

Another small change that improves the quality of life of a game developer using the nCine is the moving of headers inside an `ncine` subdirectory.
It is now very clear when you are using your game headers or the engine ones, as you would now see something in the likes of:

```cpp
#include <nctl/Array.h>
#include <ncine/Sprite.h>
#include "mygame.h"
```

Using the angle brackets makes it also easier to spot them, and they made their way into the included examples as well.

### Symbol stripping

Next is symbol stripping from binaries, a way to reduce the size of Android libraries on all supported platforms. It is also a way to strip the base libraries on ArchLinux as the `makepkg` stripping [option](https://www.archlinux.org/pacman/makepkg.conf.5.html) is not [cross-compiler aware](https://bugs.archlinux.org/task/42848) and it ends up stripping Android ARM libraries with the x86_64 `strip` command, ruining their content. :sweat_smile:

### Uniforms update

I have decided to finally spend some time identifying and squashing a long standing bug introduced with the latest renderer revamp that would cause a one frame delay when committing uniform variables. The result was very noticeable when disabling, moving and later re-enabling drawable nodes: they would be rendered in the old position for only one frame causing a very annoying graphical glitch.

At the same time I also fixed another small graphics bug that would treat 3 channels textures as gray and render them with the wrong shader.
It was evident in the texture formats example test, as it would render RGB textures in gray shades. :disappointed:

### Joystick hats

The next change is all dedicated to joystick hats, the work was in stand-by as I was waiting for the release of [GLFW 3.3](https://github.com/glfw/glfw/releases/tag/3.3). Now hats are completely separated from buttons (on GLFW with `glfwInitHint(GLFW_JOYSTICK_HAT_BUTTONS, GLFW_FALSE)`) and queried on their own on all three backends: GLFW, SDL2 and Android. When talking about joysticks one should expect the usual magic involving bit masks when working on Android, in this case to make both `AINPUT_EVENT_TYPE_KEY` and `AINPUT_EVENT_TYPE_MOTION` events update the hat state, as some joystick report one or the other type of event (see also the official Android developer article about [processing directional pad input](https://developer.android.com/training/game-controllers/controller-input#dpad)). :smile:

This change made it also possible to add the directional pad as a set of buttons when using the gamepad mapping, finally bringing it on par with the [original functionality](https://wiki.libsdl.org/CategoryGameController) in SDL2. :muscle:

### Application suspension

After some feedback by one tester I decided to rename the application pause related methods to _suspension_, a word that makes it clear that in-game pause has nothing to do with global event suspension at the application level. :relaxed:

As a bonus I have also introduced a flag that the user can set to disable the automatic suspension that happens when the focus is lost.

### Application configuration

Another small quality of life change is the removal of all the getter and setter methods from the application configuration class.
Now the user can just set a value in the configuration structure, similarly to what was already happening when changing the configuration in Lua.

```cpp
config.withVSync = false; // instead of config.enableVSync(false);
config.windowTitle = "My Game"; // instead of config.setWindowTitle("My Game");
```

### clang-format

I finally decided to commit myself to a code beautifier and stick with it. After years of playing with [Artistic Style](http://astyle.sourceforge.net/) and [Uncrustify](http://uncrustify.sourceforge.net/) without ever fully liking the results, I decided to put together a `.clang-format` configuration file.

Maybe I'm getting old and tired or maybe I'm just less scared by the stylistic compromises but I should admit I'm pretty satisfied with [it](https://clang.llvm.org/docs/ClangFormat.html). The only exception being the `IndentPPDirectives` option: I would really like to be able to use `BeforeHash` instead of manually edit the results given by `AfterHash`, but it seems I will have to wait for the [Clang 9](https://reviews.llvm.org/D52150) release. :smile:

### nCTL containers

The nCine Template Library has been given some care too, with the addition of some containers and many new unit tests (now more than 1000! :scream:) and microbenchmarks.
Most containers have seen the addition of an emplace method that uses [placement new](https://en.cppreference.com/w/cpp/language/new#Placement_new).

I have also added set containers, some are based on the hashmap ones stripped of the value associated to the key but there is also a new [sparse set](https://www.geeksforgeeks.org/sparse-set/) container that should be very fast when storing numbers and when the [space-time tradeoff](https://en.wikipedia.org/wiki/Space%E2%80%93time_tradeoff) is advantageous.

### RenderDoc integration

Next we are back with graphics, with the [in-application](https://renderdoc.org/docs/in_application_api.html) integration of RenderDoc. This means that the user can now [trigger](https://renderdoc.org/docs/in_application_api.html#_CPPv214TriggerCapturev) a capture, [launch](https://renderdoc.org/docs/in_application_api.html#_CPPv214LaunchReplayUI8uint32_tPKc) the Qt interface when running under `renderdoccmd`, disable the [overlay](https://renderdoc.org/docs/in_application_api.html#_CPPv215MaskOverlayBits8uint32_t8uint32_t), provide [comments](https://renderdoc.org/docs/in_application_api.html#_CPPv222SetCaptureFileCommentsPKcPKc) for a capture, set the [file path template](https://renderdoc.org/docs/in_application_api.html#_CPPv226SetCaptureFilePathTemplatePKc) for saving and more.

![RenderDoc in-application integration](/images/RenderDoc_integration.png "RenderDoc in-application integration")

### Buffer mapping

The last change I'm going to cover in this update is another long standing graphics issue that seems to only manifest itself on Windows 10 and Intel GPUs. There are some reproducible cases when the driver would end up displaying the first frame and doing nothing more, while the application continues to run and process events. The error does not occur when running the tests under [apitrace](http://apitrace.github.io/), [RenderDoc](https://renderdoc.org/) or [Intel GPA](https://software.intel.com/en-us/gpa), making it harder to examine.

I have discovered that the problem is related to mapping and unmapping of [uniform buffer objects](https://www.khronos.org/opengl/wiki/Uniform_Buffer_Object) and my solution so far has been to disable the feature altogether while providing an option for the user to enable it again. :weary:
When mapping is not available the renderer falls back to using a single per-frame call to `glBufferSubData()` followed by buffer orphaning, resulting in comparable performances in many situations.

And that's the end of another pretty long and detailed update, I hope you enjoyed it! :wink:
