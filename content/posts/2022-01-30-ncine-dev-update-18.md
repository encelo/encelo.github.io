---
date: "2022-01-30"
title: nCine Dev Update 18
description: Updates from the second half of June 2021 to the second half of January 2022
categories: [ Dev Update ]
tags: [ nCine, Lua, Raspberry Pi, Rendering, AppTests, Benchmarks, C.I., macOS ]
series: [ Dev Update ]
series_order: 18
---

After a very long time without an update, here comes a new one. It is filled with all the work done in the last six months. :muscle:

### Template project files alongside the engine

This was a very important change that I had in mind for quite a while.

By moving the template CMake scripts inside the engine repository, I can now update them once and reap the benefits in all nCine projects.

Often an engine update needs a change in the way projects are built, and now a single commit can logically capture that.

As the template project demonstrates, you now only need a simple [CMakeLists.txt](https://github.com/nCine/ncTemplate/blob/master/CMakeLists.txt) file, and your project is good to go!

### Lua developer distribution

I have added a new distribution option presets of the engine in the `nCine-artifacts` repository.
It should make it easier to use nCine with Lua, in a fashion similar to LÖVE.

This version only has one executable: `apptest_lua`. You can pass it an argument on the command line for the script you want to load and, thanks to a `TextNode`, it will show you on screen a message should any errors occur.

Yes, this also means you can now pass command line arguments to any of your nCine applications. :wink:

### Raspberry Pi support

I bought a nice Raspberry Pi 4 Model B with 8GB of RAM, an Argon ONE M.2 case, and a 240GB Kingston A400 SATA SSD.
The main objective was to have SpookyGhost running on it. :ghost:

![Raspberry Pi 4B](/images/RaspberryPi4B.jpg "Raspberry Pi 4B")

To achieve that I had to tinker a lot with the CMake scripts. Even if OpenGL ES was already supported to enable the Android port, I had to resolve additional issues that only came to the surface when having OpenGL ES on Linux.

I had to add a CMake script to find and check the compilation results of the atomic library on 32bit systems on ARM (Raspberry OS is a 32bit distribution), add a new `WITH_OPENGLES` preprocessor definition, and check the CMake version before using `file(ARCHIVE_EXTRACT)` (Raspberry OS comes with an older CMake version).

In the end, the result was rewarding and recognized by both [reddit](https://www.reddit.com/r/raspberry_pi/comments/qkhdus/spookyghost_my_opensource_procedural_animation/) and [Tom's Hardware](https://www.tomshardware.com/uk/news/spookyghost-comes-to-raspberry-pi). :blush:

## Viewports and cameras

The biggest achievement of this update is represented by the two months' work on viewports and cameras.

It is now possible to have multiple viewports, each one with its size, its render queue, an optional offscreen render target, an optional camera, and a particular order in the rendering chain.

The use cases are various. With the upcoming work on custom shaders you can use a viewport texture to perform full screen post-processing like blur (even with a separable filter, if you setup the viewports order chain accordingly), or you can have multiple viewports and cameras and implement a split screen!

![apptest_viewports](/images/apptest_viewports.png "apptest_viewports")

When you use a camera instead of a common camera node you can potentially improve performance as the matrix multiplications to transform nodes would be performed in a shader and not on the CPU. This can be seen in action in `apptest_camera` by pressing `V` to switch the new viewport method on and off.

There is even a greater performance gain margin when you pause sprites animations by pressing `P`. In this case, a new dirty flag system will completely skip node transformations, AABB calculations, culling rectangle intersections, and render command updates!

The dirty flags are wrapped inside a new nCTL `BitSet` container class similar to the standard `bitset` one.

The new camera system has been also ported to bigger projects like `ncTiledViewer`, `ncJump`, and JugiMap based ones.
The following tests have been performed on my Mi Notebook Pro (2017) by forcing the CPU speed to 800Mhz.

| Project Name                           | Frames (20s) | FPS    | Frame Time | Percentage |
| -------------------------------------- | ------------ | ------ | ---------- | ---------- |
| ncJugiMapFrameWorkDemo (old)           | 6387         | 319.35 | 3.131ms    | 100%       |
| ncJugiMapFrameWorkDemo (new)           | 8830         | 441.5  | 2.265ms    | 72%        |
| ncJugiMapParallaxScrollingDemo (old)   | 7168         | 358.4  | 2.79ms     | 100%       |
| ncJugiMapParallaxScrollingDemo (new)   | 8682         | 434.1  | 2.30ms     | 82.5%      |
| ncJugiMapSpriteTimelineAnimation (old) | 3537         | 176.85 | 5.65ms     | 100%       |
| ncJugiMapSpriteTimelineAnimation (new) | 5058         | 252.9  | 3.95m      | 69.9%      |
| ncJugiMapGuiDemo (old)                 | 7913         | 395.65 | 2.52ms     | 100%       |
| ncJugiMapGuiDemo (new)                 | 10691        | 534.55 | 1.87ms     | 74.0%      |
| ncJump (old)                           | 4879         | 243.95 | 4.10ms     | 100%       |
| ncJump (new)                           | 6047         | 302.35 | 3.30ms     | 80.7%      |

### Continuous integration fixes

For a long time, I have had a list of problems in [issue #11](https://github.com/nCine/nCine/issues/11) related to the GitHub Actions continuous integration.

#### macOS libraries

On macOS, the Vorbis library could not dynamically link to the OGG one because, at compile-time, the system installed one was at a different path than the one on my machine and `install_name_tool` could not perform the `@rpath` substitution.
This was recently fixed in `nCine-libraries` by modifying the Vorbis CMake building script to run `otool` to discover the embedded OGG path:

    COMMAND sh -c "install_name_tool -change $(otool -L ${FRAMEWORK_DIR_VORBIS}/${TARGET_VORBIS} | grep ${DYLIBNAME_OGG} | cut -f2 | cut -d \" \" -f1) \"@rpath/${TARGET_OGG}.framework/${TARGET_OGG}\" ${FRAMEWORK_DIR_VORBIS}/${TARGET_VORBIS}"

Thanks to Fahien and Coda for testing out the new libraries on the latest version of macOS. :pray:

#### Clang strip on MinGW

On MinGW with recent versions of Clang, the stripping of binaries fails with an obscure `unexpected associative section index` message by `llvm-strip`. I disabled it when running on this platform and opened an [issue](https://github.com/llvm/llvm-project/issues/53433) to report it.

#### GCC optimization bug

With newer versions of GCC, both on Linux and MinGW, some of the tests in the `gtest_list_movable` unit test fail in release mode. Upon further investigation, I discovered that turning down optimizations from O3 to O2 in `List.h` with `#pragma GCC optimize ("O2")` fixed the issue.
This made me think it is a compiler optimization bug, you know, one of those that goes away when you put `printf()` calls to understand what's happening. :sweat_smile:

It seems that in this particular test the `front()` method of the `List` class returns the wrong address the first time it is called, and the correct one after that. I was going to "fix" the issue by reordering the instructions in my test but then I discovered that the workaround worked on newer versions of GCC but made the error appear on older ones. :weary:

While I was creating an isolated repro I also discovered that `-O3 -funsafe-math-optimizations` was working and that not linking the GoogleTest libraries and putting the test code in a `main()` function was also working. Changing the version of GoogleTest from the latest `v1.11.0` to `v1.10.0` or to the `main` branch didn't affect the result. :disappointed:

In the end, I decided to disable the `gtest_list_movable` unit test altogether when compiling in release with GCC. Better safe than sorry!

### Minor changes

- The user has now access to the `ITextureSaver` interface to save textures in PNG or WebP formats.
- SDL 2.0.16 added a method to flash the window to request the user's attention. As this functionality was already present in the other two desktop backends, it was possible to add a generic method to the API for this to work regardless of the preferred backend used.
- The user can now use the `NCINE_WITH_SCRIPTING_API` CMake variable to enable or disable the Lua API, even when Lua utilities functions and the state manager are available.
- The standard vector and list classes are known to have undefined behavior when trying to retrieve a front or back element from an empty container. But I decided to change the nCTL versions to fatal assert in this case. It has already proven useful to fix an issue in a unit test.

I hope you enjoyed the return of development updates with this packed installment. :wink:
