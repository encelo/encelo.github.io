---
layout: post
title: nCine Dev Update 2
subtitle: Updates from August to October 2017
tags: [nCine]
---

During those three months I have been working on two big features.

The first one has been the support of SDL2 GameController [mapping](https://wiki.libsdl.org/SDL_GameControllerAddMapping) format.
Initially my plan was to build a layer on top of my joystick input functions and leave the mapping code outside, as helper functions in a file distributed along the source of my tests and only linked by them.  
Later on I decided to refactor everything and bring the code inside the engine, in an effort to make it easier for an application to transparently use it.

The mapping logic is still just a layer on top of the original joystick functions, but now the state of a mapped joystick can be queried easily from the input manager, just as you can do with the non mapped one. It is also possible for an application to listen to some new mapped events I have introduced and completely disregard the old ones.  
So, for example, instead of being notified of button 1 being pressed, the application would be notified of button "B" being pressed. This also works for axis, of course, so that an application can perform its logic on named axis, like X direction on the left stick, instead of an anonymous axis 5.

In order for the mapping to work, the joystick must be recognized. SDL2 uses a custom code per each platform in order to derive a GUID to identify the connected joystick. When compiled against SDL2 the engine can just use [`SDL_JoystickGetGUID()`](https://wiki.libsdl.org/SDL_JoystickGetGUID) and [`SDL_JoystickGetGUIDString()`](https://wiki.libsdl.org/SDL_JoystickGetGUIDString), but on GLFW things are different. It seems like next version, GLFW 3.3, is going to have full SDL2 GameController suppport and provide a `glfwGetJoystickGUID()` function, which I'm already using inside a `#if GLFW_VERSION_MAJOR == 3 && GLFW_VERSION_MINOR >= 3` block.

For previous versions of GLFW there is no GUID to return and the joystick mapping code falls back on comparing the name of the connected joystick with the one in the database, it is not completely reliable but it still mostly works.  
When running on Android, SDL 2.0.6 uses the first 16 characters of the joystick name as its GUID, a less than optimal way that just resembles closely my GLFW 3.2 fall back method. :smile: That is why I tried to do things better, especially because it is not so hard, when on API level 19, to just call the [`getVendorId()`](https://developer.android.com/reference/android/view/InputDevice.html#getVendorId()) and [`getProductId()`](https://developer.android.com/reference/android/view/InputDevice.html#getProductId()) of the `InputDevice` class and create an unique GUID with those.

I have mentioned the mappings database before, it is assembled by combining all strings from [`SDL_gamecontrollerdb.h`](https://hg.libsdl.org/SDL/file/8df7a59b5528/src/joystick/SDL_gamecontrollerdb.h), all strings from [`gamecontrollerdb.txt`](https://github.com/gabomdq/SDL_GameControllerDB/blob/master/gamecontrollerdb.txt) plus some more that are useful when running on GLFW 3.2 or on Android, given their particular GUID code.

The second big task has been the complete transition from `ndk-build` to CMake, as starting from version [3.7](https://cmake.org/cmake/help/v3.7/release/3.7.html#platforms) the developers have added the support for Android as a platform through the NDK or a standalone toolchain.  
You start by defining `CMAKE_SYSTEM_NAME=Android` and some more additional CMake variables to specify the API level, the NDK location, the CPU architecture or the STL type.  
I have decided not to use the modified CMake 3.6 that is distributed with the Android SDK, even if it is natively supported by the Android Plugin for Gradle, but the upstream latest version. This way I can support and test a single version of CMake to build on every platform.

See you again soon with new and exciting updates from the nCine development! :wink:
