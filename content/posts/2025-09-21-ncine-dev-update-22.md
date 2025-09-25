---
date: "2025-09-21"
title: nCine Dev Update 22
description: Updates from January 2025 to August 2025
categories: [ Dev Update ]
tags: [ nCine, nCTL, GGJ, University, Tools, Crashpad, Multi-Threading, AppTests ]
series: [ Dev Update ]
series_order: 22
---

Welcome back to another development update for the nCine, covering what has been accomplished in the first part of this year.

## Introspective sort

Back in December 2023, the author of Jazz² Resurrection reported a crash when sorting a render queue with more than 3000 commands. The sequence was unbalanced and quicksort recursion went too deep, overflowing the stack.

He suggested switching to introspective sort (a hybrid of quicksort, heapsort, and insertion sort) which is exactly what I ended up implementing, using the same thresholds as many standard implementations.

The algorithm sets a maximum quicksort depth at twice the base-2 logarithm of the number of elements. Up to that depth, recursive quicksort partitioning is used, which is safe for slightly unbalanced trees. If the depth is exceeded, the algorithm falls back to an iterative heapsort, avoiding stack growth. Finally, for partitions smaller than 16 elements, insertion sort takes over, as it's faster on small ranges.

## Global Game Jam 2025

I took part in the Global Game Jam 2025 in my city, where we built a game using nCine. After the jam I polished it and released it on GitHub as [Wet Paper](https://github.com/encelo/WetPaper).

Compared to the jam version, I added several features: a complete menu system, player statistics, TOML-based configuration, custom blur and refraction shaders, crossfading for music, a low-pass filter when pausing, and joystick vibration.

{{< youtube id="jvhKzdlgR4Q"loading="lazy" title="Wet Paper – Gameplay Demo (GGJ 2025 / nCine Game)" >}}

The jam also gave me a chance to improve nCine itself with new features and quality-of-life tweaks. For example, the engine now keeps track of the previous frame state for keyboard and joystick input, making it trivial to check exactly when a button was pressed or released.

I also made the vector classes return a zero vector when the length is too short (like many other engines), renamed `interval()` to `frameTime()` and `apptest_scene` to `apptest_gui` for clarity, and fixed broken alpha getters in the `SceneNode` class. A color setter override that accepted a float parameter has been renamed, and a method was added to set a `TimeStamp` to the current time without allocating a new object.

Working on custom shaders for Wet Paper also led me to fix viewport clearing logic and repair OpenGL debug groups, which hadn't been working properly for viewports.

One of the last additions was joystick vibration. SDL2's rumble API ([`SDL_JoystickRumble()`](https://wiki.libsdl.org/SDL2/SDL_JoystickRumble)) is quite barebones: you set the intensity for the two motors and a duration in milliseconds, but each call cancels the previous effect. That's why I started working on a `JoyVibrator` class to interpolate motor intensities independently. The work isn't finished yet and lives in a local `joy_vibrator` branch. :sweat_smile:

## Presentation at /dev/games

![Presenting at /dev/games 2025](https://ncine.github.io/img/posts/DevGames2025.jpg "Presenting at /dev/games 2025")

On June 5th I was invited to Rome to present my 14-year journey with nCine at the [/dev/games](https://devgames.org/) conference.

It was both a chance to catch up with friends and an opportunity to show developers and students the craft, the struggles, and the technical insights involved in sustaining a long-term project like this.

You can browse the [presentation](https://encelo.github.io/nCine_14Years_Presentation/) online. I made it with Slidev and published the Markdown source on GitHub. The talk was recorded and should appear on the official YouTube channel later this year.

### RenderDoc integration update

While preparing fresh screenshots for the talk, I revisited the RenderDoc integration and updated it to the latest 1.6.0 API. This allowed me to add features such as capture titles, automatically opening the RenderDoc UI after a capture, and enabling API validation and callstack capturing by default when using an OpenGL debug context.

## Run-time environment variables

Here's a small but useful quality-of-life change. You can now override `AppConfiguration` values at runtime through environment variables instead of recompiling your application.

This makes it easy to test your game under varying conditions: from sound frequencies and shader cache usage to window resolutions, render command pool size, or log levels.

For example:

```bash
NCINE_APPCFG_CONSOLE_LOG_LEVEL=0 NCINE_APPCFG_FILE_LOG_LEVEL=3 NCINE_APPCFG_LOG_FILE="game.txt ./my-ncine-game
```

This command disables console logging while redirecting it to a file of your choice. Combine this with scripts and you can quickly test multiple configurations. More details are available in the wiki [article](https://github.com/nCine/nCine/wiki/AppCfg-EnvVars).

## Crashpad integration

More than three years ago I began integrating Google Crashpad, a modern replacement for the now unsupported CrashRpt that I had used in `ncParticleEditor`. Crashpad is cross-platform, actively maintained by Google (they use it in Chrome), and runs as an out-of-process component, a perfect fit for my requirements.

The tricky part was Android, where distributing the `crashpad_handler` executable can trigger security restrictions depending on OS version. The solution was to ship it disguised as a JNI library and use `nativeLibraryDir()` to retrieve its location.

Another addition is the ability to extract debug info files to a user-specified directory without enabling Crashpad:

```bash
cmake -S nCine -B nCine-build -D NCINE_DEBUGINFO=EXTRACT -D DEBUGINFO_DIR=${CMAKE_BINARY_DIR}/symbols
```

This way you can upload debug symbols to platforms like Sentry, which itself uses Crashpad internally.

## Array improvements

Even the well-proven `Array` class had room for improvement.

First, I fixed a subtle bug with insertions and removals in the middle: the old implementation overwrote elements without destroying them first. Thanks to [W4RH4WK](https://github.com/W4RH4WK) on Discord for reporting it! There's now a unit test checking construction, destruction, and assignment counts for these operations.

Second, I reworked the type-trait helpers in `utility.h`. They now distinguish between trivially copyable, movable and copyable, movable-only, copyable-only, and fully non-movable/non-copyable objects.
Two new `setCapacity()` implementations now use tag dispatching to select the right behavior based on the Array class's template type. Thanks to this, you can create Array instances containing non-copyable and non-movable objects, with the restriction that arrays must be empty to resize and new elements can only be added at the back via `emplaceBack()`.
This wasn't even compiling before. :open_mouth:

## Multi-threaded job system

![Tracy capture of apptest_jobsystem](/images/Tracy_apptest_jobsystem.png "Tracy capture of apptest_jobsystem")

The star of this update is the new job system. After months of iteration, it has finally stabilized in structure and API.

Jobs now use opaque `JobId` handles that encode both a pool index and a generation number, which makes detecting stale IDs easy. I spent a long time experimenting with a fully lock-free job pool but couldn't get correct behavior. The final design uses per-thread job caches and a global pool protected by a mutex.

Worker synchronization now uses semaphores instead of a mutex + condition variable pair to wake threads when new jobs are available. This avoids the extra lock/unlock overhead and makes the wakeup path more direct. On each platform the fastest available primitive is used: on Linux a futex-based userspace semaphore, on Windows [`WaitOnAddress()`](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitonaddress) with atomics instead of a kernel semaphore object, and on macOS a Grand Central Dispatch semaphore instead of a pthreads implementation.

To tame the many threading issues during development, I added `jobsystem_debug.h`, which lets you enable Tracy zones and plots, statistical counters, additional logs, and job state tracking via compile-time flags. These debug tools have been lifesavers.

There's also a serial job system: if you set a single-thread mode in `AppConfiguration` you'll get the same API without synchronization, perfect for debugging or measuring scalability.

A recent commit also added a handle class with an object-oriented API over job IDs, new job state flags to prevent double submissions and support cancellation, and a new submit call to allow for multiple jobs to be submitted at once.

### CPU topology

![CPU Topologies Diagram](/images/CPU_Topologies.png "CPU Topologies Diagram")

I completely reworked how the thread pool is created and how affinity is set, making the system topology-aware.

The idea is to sort cores by speed, to leave the main thread unpinned, and to start pinning worker threads from the second fastest physical core.

I have tested it on some of my devices:

- On a Ryzen 9 6900HS (8C/16T), it creates 7 workers pinned to physical cores, leaving the main thread free.
- On an Intel i5-1235U (2P + 8E, 10C/12T), it creates 9 workers: one on the second performance core and eight on the efficiency cores, with the main thread unpinned.
- On a Mac Mini M1 (4P + 4E), it pins 7 workers, leaving the main thread unpinned and the first performance core free.
- On a Snapdragon 8 Gen 3 (1Pr + 3P + 2P + 2E), it uses sysfs `cpu_capacity` values to pin 7 workers across the three weaker clusters, leaving the main thread unpinned and the Prime core free.

This could eventually allow more advanced scheduling in the future, with certain job tags mapped to faster or slower cores depending on priority.

### Conclusions

The best part: the job system is fully exposed to applications. You can try it right now in the new `apptest_jobsystem` test. :raised_hands:

The `job_system` branch is already live and up to date, though it needs more testing before merging. Meanwhile, I've begun work on a data-oriented ECS, an ideal testing ground for the job system. More on that in the next update.

## Minor changes

- Added conversion functions between `Vector*i` and `Vector*f`, and between `Recti` and `Rectf` classes
- Updated `README` with documentation links and screenshots :camera:
- Added new dirty bits to nodes (thanks to [Jugilus](https://github.com/jugilus)) to avoid redundant transformations of culled nodes
- Fixed a long-standing bug with string capacity changes when using custom allocators
- Updated GitHub Actions runners: added macOS 15 and Ubuntu 24.04, retired VS2019
- Vector, matrix, and quaternion classes now have inequality operators :smile:
