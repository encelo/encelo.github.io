---
layout: post
title: Get to know the nCine
tags: [nCine]
---

I am writing this to present my latest and biggest project to date, the nCine.
I was originally going to publish an article the day that I was going to release the source code but for various reasons you will have to wait a bit more. :cry:  
I am nevertheless going to describe today some of the technical aspects behind it hoping that when it’s ready for release there would already be someone interested in using it. :wink:

[![nCine banner](/img/ncine_banner.png "The nCine banner")](https://ncine.github.io)

nCine is a multi-platform 2D game engine that works both on PC (meaning Linux, Windows and OS X) and on Android. Yes, the name is a portmanteau of my nickname “Encelo” and the word “engine”. :sweat_smile:  
I started working on it in June 2011, at the time I was using a Mercurial repository to keep track of changes but later on I converted it to a Git one. Some of the developing tools I use all the time are [Qt Creator](https://www.qt.io/ide/) as my IDE of choice (plus the Community Edition of [Visual Studio](https://www.visualstudio.com/vs/community/) on Windows), [CMake](https://cmake.org/) for everything related with the building and packaging phases, [Doxygen](http://www.doxygen.org/) and [Graphviz](http://www.graphviz.org/) for the documentation, [Cppcheck](http://cppcheck.sourceforge.net/) for the static analysis, [Valgrind](http://valgrind.org/) for additional memory debug and [Artistic Style](http://astyle.sourceforge.net/) for the automatic formatting.

The project has a wide support in compilers as well, GCC and LLVM on Linux, MSVC and GCC (via [MSYS2](http://www.msys2.org/)/MinGW-w64) on Windows, LLVM on OS X and GCC and LLVM on Android with the NDK. At the moment there are more than one and a half thousand lines of CMake scripts and more than twenty five thousand of C++ code, excluding comments and blank lines. :muscle:  
The nCine is a framework of classes and can be built as a static or dynamic library.

This work has always been intended as a way for me to learn new things and develop my skills as an engine programmer, this is the reason why I have implemented many things from scratch instead of relying on external libraries.  
For example I don’t use the STL but I have my own implementation of template based containers (list, array, hashmap, string, …), algorithms and iterators. It was a very good way to overcome the fear of templates and learn about type traits, concepts, tag dispatching or SFINAE. :smile:  
The project also gave me the opportunity to learn low-level OS APIs such as monotonic timers, threads and their synchronization and affinity. All those primitives are handled differently by the Linux, Windows and Mach (OS X) kernels.

<iframe width="640" height="360" src="https://www.youtube.com/embed/fUNGf3C8SOM" frameborder="0" allowfullscreen></iframe>

There are still, of course, some external dependencies: for the rendering I am relying on OpenGL, capable of reaching all the OSes I’m targeting, while for the sound I have used [OpenAL Soft](http://kcat.strangesoft.net/openal.html). On top of the latter I use [Ogg](https://www.xiph.org/ogg/) and [Vorbis](https://www.xiph.org/vorbis/) to enable music streaming and playing compressed sound samples (with uncompressed Wave being another supported option). For interfacing with the window and the input system I use both [GLFW](http://www.glfw.org/) 3 and [SDL](https://www.libsdl.org/) 2, depending on which one is available, while on Android I use EGL and the Android API, sometimes through the NDK API and sometimes, like when accessing the joystick, directly through JNI calls.  
For texture loading I use [libpng](http://www.libpng.org/pub/png/libpng.html) and [WebP](https://developers.google.com/speed/webp/), but the engine is also perfectly capable of loading GPU compressed formats such as DXT, ETC1, ATC, PVR and ASTC, embedded in DDS, [KTX](https://www.khronos.org/opengles/sdk/tools/KTX/) or PVR texture container formats.  
When it comes to font rendering I have written a parser for AngelCode’s [BMFont](http://www.angelcode.com/products/bmfont/) FNT format so that the engine can render text strings with support for [kerning](https://en.wikipedia.org/wiki/Kerning) pairs and customizable horizontal alignment between multiple lines.

The dependencies that I have just mentioned are compiled directly from upstream sources for all supported platforms thanks to a set of custom CMake scripts. I am able to create the .so shared libraries on Linux, the DLLs on Windows, the frameworks on OS X and cross-compile for all the supported Android architectures (armeabi-v7a, arm64-v8a and x86_64). The scripts are also responsible for putting together the NSI installer and the ZIP portable archive on Windows, the application bundle on OS X and the TAR.GZ portable archive on Linux. For Arch Linux and MSYS2, which are both based on Pacman, I have written the corresponding PKGBUILD scripts to create a compressed package.

I will soon release the engine on GitHub under a MIT license. At the moment you can have a look at the [website](https://ncine.github.io/), which contains the initial API [documentation](https://ncine.github.io/docs/), and you can follow the project on [Twitter](https://twitter.com/ncine2d).

I hope to come back often in the future to write about new features and very soon to announce a full release!
