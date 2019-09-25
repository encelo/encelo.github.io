---
layout: post
title: nCine Dev Update 10
subtitle: Updates from the second half of May 2019
tags: [nCine]
---

I'm sure many of you have heard it already: the nCine source code has been released on [GitHub](https://github.com/nCine)!

This means that lately most of the time was dedicated to publication related tasks, for example updates to the site like the addition of a "[why nCine?](https://ncine.github.io/why/)" page and a [gallery](https://ncine.github.io/gallery/).

But definitely one of the most complex task has been continuous integration, which has a new [page](https://ncine.github.io/ci/) on the site too. :wink:

### Continuous Integration

A couple of years ago I began experimenting with [Travis](https://travis-ci.org/) and [AppVeyor](https://www.appveyor.com/) and managed to have them build the libraries, the engine and ncPong for all supported platforms: Linux, Windows (both MSVC and MinGW), macOS and Android.

In an effort to simplify this setup I decided to migrate everything to [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/).
It supports all major desktop systems meaning I now need only one script to support everything. It also features a more flexible YAML language that organizes the whole process as a sequence of jobs and steps with conditions, as opposed to specific steps like `after_test` or `before_install`.

Now that the source code is available online, let's have a look at those scripts.

#### Building the libraries

First thing to do is to build the dependencies, both for [desktop](https://github.com/nCine/nCine-libraries/blob/master/azure-pipelines.yml) and for [Android](https://github.com/nCine/nCine-android-libraries/blob/master/azure-pipelines.yml).
As you can notice libraries are built on Linux with both GCC and clang and on Windows with both VS2019 and VS2017. On Android libraries are built for the three supported architectures but only on Linux.

Next is when things become interesting, an idea that I kept from my original tests in 2017. In order to have artifacts available regardless of the C.I. platform and available for users to download I created some repositories on GitHub with the sole purpose of storing them.

For libraries the repository is named [nCine-libraries-artifacts](https://github.com/nCine/nCine-libraries-artifacts) and is made of multiple [branches](https://github.com/nCine/nCine-libraries-artifacts/branches/all), each one for a specific platform.

#### Building the engine

Next is the engine itself, it pulls the library artifacts and engine data and push installers and portable archives.
The [script](https://github.com/nCine/nCine/blob/master/azure-pipelines.yml) gets more complicated by a test matrix that now includes build types, like *Debug* and *Release*, together with the *DevDist* preset used for installers.

The first steps prepare the environment, they download [Doxygen](http://www.doxygen.nl/) and [Graphviz](https://www.graphviz.org/) to build the documentation and the NDK to compile for Android.
On Windows and Linux they also download the RenderDoc API header because CMake is invoked with `-D NCINE_WITH_RENDERDOC=ON`.
Additional steps will build unit tests and run them through `CTest`. Benchmarks will also be built but not executed.

At the end packages are created via the `package` CMake target and files pushed in the [nCine-artifacts](https://github.com/nCine/nCine-artifacts) repository. It will again have multiple [branches](https://github.com/nCine/nCine-artifacts/branches/all) depending on platforms but this time they will also also depend on engine source branches.

#### Building the projects

Now that the engine has been built and its artifacts pushed on GitHub it is the turn of the accompanying projects.
Both the [pong](https://github.com/nCine/ncPong/blob/master/azure-pipelines.yml) example game and the [particle editor](https://github.com/nCine/ncParticleEditor/blob/master/azure-pipelines.yml) have very similar C.I. scripts.

They download the libraries, the engine and the project data and they build for all supported platforms, including Android APKs using Gradle.
They push artifacts in the [ncPong-artifacts](https://github.com/nCine/ncPong-artifacts) and [ncParticleEditor-artifacts](https://github.com/nCine/ncParticleEditor-artifacts) GitHub repositories.

#### Some notes

- On Linux CMake is not very recent and a special step has to download and install an updated version from the official site.
- On Windows all steps use [PowerShell](https://Microsoft.com/PowerShell). Life is too short to mess with the Command Prompt. :sweat_smile:
- Often enough git commands write their output to `stderr` and make scripts fail. That's the reason for `$env:GIT_REDIRECT_STDERR = '2>&1'` in PowerShell steps.
- On PowerShell `curl` is an alias to an internal `Invoke-WebRequest` command which does not understand all curl options. The solution is `Remove-item alias:curl`.
- Sometimes there is a need for git commands to invoke a fallback when they fail, it's achieved this way:
  - `git fetch --unshallow || true` on Bash
  - `git fetch --unshallow; if (-not $?) { return }` on PowerShell
- MSYS steps always set the `CHERE_INVOKING` environment variable for Bash to use the current working directory.

### More changes

Continuos integration has been a good way to spot some tricky issues occurring only with specific combinations:
- Tracy memory profiling not working on macOS
- Wrong atomics version used on MinGW
- Unit tests and benchmarks not compiling on MSVC
- Wrong version of OpenGL headers included on macOS
- Static library support breaking in some situations

Fortunately in the end it wasn't all about C.I. and I had some time for a small new feature: a configurable frame limiter.
It might come in handy especially on mobile to limit FPS below VSync. :muscle:
