---
date: "2019-04-01"
title: nCine Dev Update 8
description: Updates from March 2019
categories: [ Dev Update ]
tags: [ nCine, Android, CMake ]
series: [ Dev Update ]
series_order: 8
---

I have spent some very intense weeks this March to completely overhaul and refactor my CMake scripts. We are talking about more than two thousand lines of code! :scream:

### Android

I started by changing the way I cross-compile on Android. I dropped the use of CMake [integrated NDK support](https://cmake.org/cmake/help/v3.7/manual/cmake-toolchains.7.html#cross-compiling-for-android-with-the-ndk) introduced with version 3.7 and went back using the [toolchain file](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html#cross-compiling) provided by the NDK itself. It seems [the way to go](https://github.com/android-ndk/ndk/issues/463) in order to provide full control and flexibility to each NDK update.

I have also used this opportunity to add support for NDK r19 and its embedded [stand-alone toolchain](https://developer.android.com/ndk/guides/standalone_toolchain), the new [3.x Gradle plugin](https://developer.android.com/studio/releases/gradle-plugin#3-0-0) and LLVM's [`libc++_shared`](https://developer.android.com/ndk/guides/cpp-support#libc).

### Target properties and expressions

I then started to broaden the use of more target related CMake commands, as [`target_sources`](https://cmake.org/cmake/help/latest/command/target_sources.html) or [`target_link_options`](https://cmake.org/cmake/help/latest/command/target_link_options.html). This last command raised the minimum CMake version to [3.13](https://cmake.org/cmake/help/v3.13/release/3.13.html).

Taking full advantage of CMake targets allowed me to completely remove any `set` command involving variables like `CMAKE_CXX_FLAGS` or `CMAKE_SHARED_LINKER_FLAGS`.
Besides that I also started to rely more deeply on [generator expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html) for things like `$<TARGET_FILE:tgt>`, `$<BUILD_INTERFACE:...>` or `$<CONFIG:cfg>`. This last expression got rid of every `if(CMAKE_BUILD_TYPE MATCHES "Debug")` conditional check. :sunglasses:

### Imported targets

But I didn't stop there and I went on to convert my scripts to use targets everywhere. I now use [imported targets](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets) for every external dependency library.
In the end it's a better and more flexible way of handling the problem but it put a lot of stress on testing as not all `find_package` scripts correctly set targets and some platforms need additional tweaking too.

- For example MSYS/MinGW needed a custom CMake function to find the DLL and set the `IMPORTED_LOCATION` target property as the corresponding `find_package` scripts only discovered the import library.
- As another example on macOS I needed a different custom CMake function to split the link options set by `find_package` scripts into the ones representing framework directories and the ones being `-framework` link directives.
There is also a need to append the library symlink name to the framework directory as that is what the `IMPORTED_LOCATION` target property [expects](https://cmake.org/cmake/help/v3.14/prop_tgt/IMPORTED_LOCATION.html) when dealing with macOS frameworks (someone has even opened an [issue](https://gitlab.kitware.com/cmake/cmake/issues/18753) ticket about that).

### Discoverability and exported targets

In an attempt to make nCine discoverability more robust for external projects I have changed a lot of code based on cascaded conditions to a bunch of a lot cleaner `find_path` calls.

I now also always use [exported targets](https://gitlab.kitware.com/cmake/community/wikis/doc/tutorials/Exporting-and-Importing-Targets#exporting-targets) and this has two very important benefits:

- It is now possible to link to the `ncine` target and automatically have access to interface properties like for example [`INTERFACE_INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html).
- By using the `nCine_DIR` CMake variable when configuring a project it is possible to import the targets from a build directory.

The outcome of this last change is that there is no more `NCINE_HOME` custom variable, all the discovery use cases are handled by the CMake way of doing things: using a `<package>_DIR` variable that points to where the `nCineConfig.cmake` script is. This will in turn include the `nCineTargets.cmake` script to bring all the imported nCine targets into the namespace of an external package `CMakeLists.txt` file.

### Bonus

A couple of additional changes are:

- I now build a new `ncine_main` static library and export it as a target, instead of distributing a `main.cpp` file that the user never really had the need to modify.
- I have added an [interface library](https://cmake.org/cmake/help/v3.14/manual/cmake-buildsystem.7.html#interface-libraries) pseudo target for the code coverage compiler and linker options as a way, as the CMake manual states, "_to employ an entirely target-focussed design for usage requirements_". :muscle:

That's all for this update, I hope you enjoyed this in-depth review of CMake changes.
