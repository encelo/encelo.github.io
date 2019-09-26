---
layout: post
title: nCine Dev Update 12
subtitle: Updates from the second half of July to the first half of September 2019
tags: [nCine]
---

The last two months of work on the nCine were mostly dedicated to the quality of life improvements for users.

First of all, I decided to get rid of the legacy debug overlay. It was a very old and problematic code that didn't have any reason to be today. With the ImGui and Tracy integrations in place, the nCine is more than covered in that aspect. :muscle:

In an attempt to make the use of `nctl::string` less confusing for users of `std::string` I refactored and renamed many `copy` methods to `assign`.

### FNT parser

While I was explaining to a user how to make the font in his game bigger by creating a new texture, I suggested [fontbuilder](https://github.com/andryblack/fontbuilder), a multi-platform tool I found on GitHub. I then tried the tool myself and when I used the generated texture the text was not properly rendered in the nCine. :sweat_smile:

I knew the [FNT format](http://www.angelcode.com/products/bmfont/doc/file_format.html) has different ways to encode glyph data but I was supporting only a couple of them.
I took some time to rewrite my `FNT` parser with the specifications in mind, extracting all the available data allowing me to add some additional encoding formats.
Most importantly it provides the engine with the reasons why a specific encoding is not supported, leading to more meaningful error messages.

### Timer refactoring

I refactored the `Timer` class to be just an interface to a stopwatch. The platform specific code to access the monotonic clock and retrieve the elapsed ticks is now inside a `Clock` class.

Besides using the timer, the user can now also use the `TimeStamp` class to record or accumulate counter values, with the possibility to convert them to various units of measure in a way similar to [std::chrono::duration](https://en.cppreference.com/w/cpp/chrono/duration).

### Tracy v0.5

With the release of [Tracy v0.5](https://bitbucket.org/wolfpld/tracy/src/v0.5/) last August it was time to update its integration inside the engine and make it easier for external projects to use it.

Additionally, you can now rename a thread to easily spot it in the main profiler window and access all engine log entries in the messages section.
Good news for macOS users, memory profiling is now working and has been re-enabled. This new version of Tracy should also be able to compile on MinGW.

### Windows changes

The Windows port was given some care, with the removal of every `Windows.h` inclusion and the [minimization](https://aras-p.info/blog/2018/01/12/Minimizing-windows.h/) of the headers used.
The atomic implementations were moved from headers to sources, removing the need for including a Windows API header in `<nctl/Atomic.h>`.

Two more Windows improvements are the use of the Windows subsystem instead of console for apptest executables and the installation of a desktop link by the NSIS generated installers.

Last but not least the engine and the game projects now generate a [VERSIONINFO resource](https://docs.microsoft.com/en-us/windows/win32/menurc/versioninfo-resource) file.
Thanks to it you can open the properties window of files like `ncine.dll` or `ncpong.exe` and retrieve product and version information.

![VERSIONINFO resource](/images/VERSIONINFO.png "VERSIONINFO resource"){: .center-block :}

### Granular library dependencies

Some users didn't like Lua at all and asked me how to remove the bindings integration. :smile:
I told them this was totally possible as the nCine was modular since its inception: you could already take out library dependencies at the expense of functionality.
The problem was that the feature was not easily accessible to users and properly tested among all combinations.

Now it is and you can turn off the `NCINE_WITH_LUA` CMake option to disable the integration. The engine will not be linked with the Lua library and not packaged with it in installers or archives. There are similar options to disable threads, audio or the integration with decoding libraries such as `libpng`, `libwebp` or `libvorbis`.

### Additional projects

A lot of work was devoted to adding or improving accompanying projects too, in an attempt to further ease the life of nCine users.

For example, a new [ncTemplate](https://github.com/nCine/ncTemplate) repository was pushed to GitHub. It is supposed to be the starting point for creating a new project with the engine.
By using it you ensure that all the CMake logic to support all platforms is in place.
Check the [README.md](https://github.com/nCine/ncTemplate/blob/master/README.md) file for additional information.

There is a second repository that has been created to make working with the nCine easier: [ncline](https://github.com/nCine/ncline), the nCine command line tool.
It all started a couple years ago when I thought a Python script could automate some of the tasks related to calling CMake on different platforms.

That's when `ncDo.py` was born, an internal tool I used to speed up the compilation and testing of the nCine when switching from an operating system to the other.
Today the _continuous integration_ helps a lot in that regard but an automation tool is still very valuable for me and for users.

I decided to rewrite it in C++ as it was growing more complex and I wasn't comfortable with Python anymore. Thanks to [clipp](https://github.com/muellan/clipp) and [cpptoml](https://github.com/skystrife/cpptoml) I managed to keep all the features that were once carried out by [argparse](https://docs.python.org/3/library/argparse.html) and [configparseer](https://docs.python.org/3/library/configparser.html).
The tool has now more functionalities than ever before, being able to download source code or artifacts using Git.
Just like earlier, have a look at the [README.md](https://github.com/nCine/ncline/blob/master/README.md) file for more information.

Last but not least, I made a first GitHub [release](https://github.com/nCine/nCine/releases/tag/2019.05) and started [keeping track](https://ncine.github.io/download-develop/) of differences between releases and the `develop` branch. Yet another small quality of life improvement, especially for users migrating to a newer version.

I hope those changes will make the nCine easier to use to create awesome games! :wink:
