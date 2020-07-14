---
layout: post
title: nCine Dev Update 14
subtitle: Updates from the second half of December 2019 to the first half of March 2020
tags: [nCine]
---

Welcome to another nCine development update! As usual, there are a lot of new things to cover.

### ANGLE

To extend the support to more devices and platforms I have ported the nCine to [ANGLE](http://angleproject.org).
It was an easy task as I just needed to tell GLFW and SDL2 to use EGL and open an OpenGL ES 3.0 context on Windows.

With ANGLE the application would use Direct3D 11, overcoming any possible bug in old OpenGL drivers on old GPUs. It would also enable a future porting to [UWP](https://en.wikipedia.org/wiki/Universal_Windows_Platform).

You have to sacrifice some performance, unfortunately. On my Intel i7-8850u with UHD 620 GT2 the `apptest_camera` test is capable of rendering nearly 1.5x frames in 5 seconds when using the OpenGL driver instead of ANGLE.

| Version    | Batches size |  Frames | Ratio |
|------------|--------------|--------:|------:|
| OpenGL 3.3 | unlimited    |    6263 | 1.461 |
| OpenGL 3.3 | 10           |    6006 | 1.401 |
| ANGLE      | 10           |    4286 | 1.000 |
| OpenGL 3.3 | disabled     |    4819 | 1.432 |
| ANGLE      | disabled     |    3365 | 1.000 |

When buffer mapping is enabled the performance takes a huge hit. :persevere:

| Version    | Batches size |  Frames | Ratio |
|------------|--------------|--------:|------:|
| OpenGL 3.3 | unlimited    |    5559 | 2.871 |
| OpenGL 3.3 | 10           |    5302 | 2.738 |
| ANGLE      | 10           |    1936 | 1.000 |
| OpenGL 3.3 | disabled     |    4069 | 1.432 |
| ANGLE      | disabled     |     974 | 4.177 |

### Qt5 Backend

I have added [Qt5](https://www.qt.io/) as a new desktop backend.

This will open more possibilities for tool developers, as they can now create a Qt5 interface and embed the nCine as a custom [QOpenGLWidget](https://doc.qt.io/qt-5/qopenglwidget.html).

Using Qt5 for the interface does not mean you can't continue to use ImGui and Nuklear too. You can even use all three together if you so want, as the updated `apptest_scene` demonstrates. :wink:

<div class="embed-responsive embed-responsive-16by9">
  <iframe width="640" height="360" src="https://www.youtube.com/embed/PpVLD3ShiCw" frameborder="0" allowfullscreen></iframe>{: .center-block :}
</div>

If the [Qt Gamepad](https://doc.qt.io/qt-5/qtgamepad-index.html) library is found then gamepad events will be supported. I also decided to expose touch events to desktop applications, as both SDL2 and Qt5 support them.

### FileSystem API

Another big addition is the new file system API. It allows an application to manipulate paths, query file and directory attributes, copy, rename or delete files, create directories and inspect their contents and so on.

It works on all supported platforms as it features both a POSIX and a Windows API implementation.

It comes with a unit test that should assure everything works as expected and with a new apptest. `apptest_filebowser` displays a file selection window made with ImGui that makes it easy to browse the file system.

![apptest_filebrowser](/images/apptest_filebrowser.png "apptest_filebrowser"){: .center-block :}

### Emscripten

I fixed the compilation with Emscripten version [1.39.5](https://emscripten.org/docs/introducing_emscripten/release_notes.html) that makes `DISABLE_DEPRECATED_FIND_EVENT_TARGET_BEHAVIOR` the [default](https://groups.google.com/forum/#!msg/emscripten-discuss/xScZ_LRIByk/_gEy67utDgAJ).

It means it is not possible anymore to pass a `nullptr` as a target in the [HTML5](https://emscripten.org/docs/api_reference/html5.h.html) API and expect it will automatically choose a reasonable element. You have to be explicit and pass things like `"canvas"` or `EMSCRIPTEN_EVENT_TARGET_WINDOW`.

### Minor Changes

* As it is sometimes useful, it is now possible to multiply a vector by a matrix with the matrix on the right side of the multiplication.

* C-style strings can now correctly be used as hashmap keys. Previously the key comparison code was comparing memory pointers instead of characters, making this type of strings useless with hashmap containers.

* If you compile the engine statically you will have access to a couple of classes that save textures as PNG or WebP images.

It is all for now. See you for the next update! :wink:
