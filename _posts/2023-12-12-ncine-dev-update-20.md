---
layout: post
title: nCine Dev Update 20
subtitle: Updates from August 2022 to November 2023
tags: [nCine]
---

Lately, the development rate of the nCine slowed down a bit. I think it is normal for a project that spans so many years, and developed by a single person, to see some oscillations.
This is why this article covers such a long period, a period in which there have been maybe a few new features, but important ones.

## Binary shaders

For sure the most interesting, and time-consuming, feature added since the last article is the support for binary shaders.

When it is enabled, OpenGL shaders are compiled only the first time an application runs. They are then cached on disk as binary blobs and loaded on subsequent runs.
Loading a binary file is, of course, a lot faster than loading and compiling sources.

The feature was developed mainly for ANGLE, as its OpenGL to DirectX shader translation and compilation layer is quite slow. ANGLE is used on Windows inside browsers, but WebGL disables loading and saving binary shaders for security reasons.
The main interest in making ANGLE shaders load faster resided in [Jazz² Resurrection](https://github.com/deathkiller/jazz2-native) support for Universal Windows Platform and thus Xbox. :video_game:

The feature is enabled by default on Android as it can help slow devices to start an application faster.

#### Hashing

A fundamental part of this new feature is hashing shader sources. If shader sources have not changed, we can continue to load the same binary file, otherwise, we need to compile and save a new one.

I was initially hashing all the sources used by a shader, every time I was loading it. This leads to correct results all the time, of course, but it can be slow and superfluous, especially for the default shaders that come with the nCine.

Default shader sources can be embedded in the nCine library or loaded from files, in both cases hashing the sources at run-time is not needed. In the first case, I use CMake to calculate an MD5 hash of source contents and embed it in the library. In the second case, I calculate a custom hash value from the name, size, and modification date of the file.

The hash is then compared with the value in a shader info file, a text file that contains a list of all successfully compiled shaders.

While the hash calculated by sources is a very important value to identify a particular shader, there is also another hash value that is combined to create the binary shader filename: the platform hash. It is calculated from the `GL_RENDERER` and the `GL_VERSION` strings and it ensures that, if the OpenGL driver is updated, shaders will be recompiled.

The hashing methods for strings and files are part of a new `Hash64` class that internally uses `fasthash64` and provides a Lua API.

#### Maximum batch size

Not related to binary shaders, but implemented while revamping the shader compilation pipeline, is the double compilation of batched shaders. Under certain circumstances, for example when the maximum supported size for Uniform Buffer Objects is less than 64 Kb, a batched shader is compiled a first time with a batch size of one so that its size requirements can be queried.

It is then compiled a second time with the batch array perfectly sized to match the available UBO maximum size. This feature improves the compatibility on some Android devices that do not support the classic 64 Kb UBO size and failed the compilation. In all other cases, the batch size value in shader sources is adjusted for a 64 Kb UBO and the shader does not need to be compiled two times.

While testing this feature I also encountered a bug in the `RenderBatcher` class that could have caused issues with small UBOs and big batches: basically, the condition that forced a split in a batch upon finishing UBO space was wrong. :sweat_smile:

## HiDPI support

Another quite important feature added last year was the support for HiDPI displays.

The first step towards this goal was extracting as much information as possible from the backends. This is why multiple monitors are now supported.

Based on the backend, it is now possible to know in real-time if a monitor has been connected or disconnected and to move a window from one monitor to another.
Even on Android, using some JNI glue code, it should be possible to handle monitor connection and disconnection events.

I have also added the possibility to specify a particular refresh rate on init, for when the application starts in fullscreen.
The user can also specify a window position so that the application can start on the specified monitor and at the specified position.

But supporting HiDPI also means adapting the window size and its contents to the scaling factor applied by the operating system. This feature is demonstrated by the new `apptest_scaling` application.
It uses the new `onChangeScalingFactor()` callback to detect if the user explicitly changed the scaling factor of the monitor or dragged the application window to a different monitor.

Unfortunately, even after a long period of testing, the feature does not seem to be working perfectly, especially on Emscripten. I hope things will get better with new versions of the backends.

### Thread class move fixes

The `Thread` class was developed years ago and tested only marginally, it is only normal it had some hidden bugs.

One of these was discovered by the Jazz² Resurrection author and happened when a thread object was moved into another one. Sometimes the source object went out of scope before the OS scheduled a new thread, and the pointer passed to the OS API function became invalid, causing a crash.

To fix this, I wrote custom move special member functions and a swap method, and I moved the threading information structure inside a `UniquePtr` so that it is allocated on the heap and moved correctly.

I also used the opportunity to rename some internal variables, change some method signatures to return a success boolean flag, and add a new `detach()` method.

### Bunnymark

[Bunnymark](https://github.com/openfl/openfl-samples/tree/master/demos/BunnyMark) is a famous 2D benchmark that has been implemented in many different engines and frameworks to showcase the rendering and game logic performance.

I have implemented a nCine version, `apptest_bunnymark`, using the same sprite and logic from the original one. I have also improved the frame timer class and it is now more powerful and flexible.

Unfortunately, the performance isn't yet where I would like them to be, but the benchmark will be an additional tool to help make the nCine even faster! :rocket:

### Improvements to particle systems and affectors

There have been some minor improvements to the particle systems and affectors. For example, affectors can now add steps even if specified out of order, instead of rejecting them. They also come with a new Lua API, so that complex systems can be built with scripting.

I have also resolved a minor issue when initializing a new particle system with random parameters: they had to respect certain numeric constraints or the system would not work as expected (or even assert and exit :bomb:). Now the constraints are not only checked but enforced by clamping or reordering values.

To have more flexibility with complex systems, there is now the option to disable a specific affector or to completely disable the particle update.

### Minor Changes

- Long error textnodes in `apptest_lua` are now scaled down to be readable in their entirety
- There is a new method to add multiple rectangles to an animation at once
- Two new CMake presets have been added to support unit tests and micro-benchmarks use cases
- The `apptest_lua` application will detect any changes to the `script.lua` file and hot-reload it automatically
- The `audio`, `font`, and `particles` apptests have now an ImGui debug interface
- The Android JNI code should now be more stable by using global references
- All input backends have now the support for a "_drop files_" event
- The application will never exit when a file cannot be opened
- You can now choose to render `TextNode` objects by using the fragment shaders of regular sprites
- Android gamepad code has been revamped by supporting a fallback system mapping, more button strings, and mapping buttons as axes
