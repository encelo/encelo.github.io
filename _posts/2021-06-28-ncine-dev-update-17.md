---
layout: post
title: nCine Dev Update 17
subtitle: Updates from the second half of November 2020 to the first half of June 2021
tags: [nCine]
---

Quite some time has passed since the previous development update but I'm here again to talk about the latest nCine progress.
By the way, in case you missed the latest [article](/2021-06-21-ten-years-ncine/), the project has recently reached its tenth anniversary. :wink:

### New nCine projects

Many of the following fixes and features have been driven by the work on some new nCine projects that have seen the light of day during those months:

- [ncTiledViewer](https://github.com/nCine/ncTiledViewer) is a loader and viewer of [Tiled](https://www.mapeditor.org/) maps that you can easily integrate into your game.
- [ncJump](https://github.com/Fahien/ncJump) is a platform game from Fahien. He has integrated Box2D physics to create one of the most complex nCine projects to date! :pray:
- [SpookyGhost](https://github.com/SpookyGhost2D/SpookyGhost), my procedural sprite animation tool, has recently become free and open source. I took advantage of this opportunity to add many new features.

### Default constructors for nodes

As I was anticipating in the last update, I pushed the resource system even further by having nodes that can initially be constructed without any resource assigned to them.

Think for example of a bunch of sprites in an array that are constructed via their default constructor. They will have no texture or parent but they can still have a color.
Solid color sprites will use a specific shader that doesn't sample any textures at all. They will be batched together and render faster than regular textured sprites.

But it does not end here: animated and mesh sprites can both be default constructed and not be associated with a texture. Text nodes can be default constructed too and particle systems can have texture-less particles.

This functionality has been extended to audio buffer players as well, they can be default constructed as well.

You can of course assign a resource to a default constructed node at a later time.

### Colored console messages

This feature was requested by Fahien while he was working on ncJump and I couldn't help but satisfy the user working on the most ambitious project. :wink:

So not only I added this feature but I fixed the lack of a console window on Windows, an issue plaguing this platform since I changed the default subsystem.

Now we can have the windows subsystem and the `WinMain` function, and open a console if the user wants it or if we are running a debug build.
This is possible by using the [AttachConsole](https://docs.microsoft.com/en-us/windows/console/attachconsole) API function.

As a plus, I have ported from `ncline` the code to set the `ENABLE_VIRTUAL_TERMINAL_PROCESSING` and have color output in the Windows command prompt. :muscle:

![Colored Console Messages](/images/colored_console_messages.png "colored_console_messages"){: .center-block :}

### Fault-tolerant Lua scripts execution

The most important new feature of this update is probably the new way of running Lua scripts. In the past, an error in a Lua script caused an assertion that killed the application.
This is, of course, not acceptable. Users should be able to safely play with scripts, either in games or tools. For example, writing scripted animations in `SpookyGhost`.

The main change was to replace each `lua_call()` with `lua_pcall()`. The latter works in protected mode and returns an error in case the script is malfunctioning.

The error message is optionally returned to the application with additional information extracted from a `lua_Debug` structure. The user can use it to show where the error is in the script source.

![SpookyGhost Script Error](/images/SpookyGhost_script_error.png "SpookyGhost_script_error"){: .center-block :}

I have also removed all occurrences of `luaL_argerror()` and replaced them with simple warning messages to avoid any exit caused by this kind of error.

To give the user even more flexibility I made it possible to load a script without running it immediately: loading a Lua "chunk" and calling it are now two separate actions.

As a bonus, I also fixed the parameter order of all calls to `lua_createtable()`. :sweat_smile:

### New application and input events

#### Text input event

Reacting only to keypresses is not enough in the world of Unicode and UTF8, as text characters are not the same as pressed keys.
For this reason, I have exposed to the user a new input event that should make it easier to process text: `onTextInput()`.

SDL2 already came with an `SDL_TEXTINPUT` event and GLFW with a `CharCallback`. They work slightly differently but wrapping them was easy enough.

For Qt5 I check the length of the string returned by `QKeyEvent::text()` and generate the event if it's longer than zero. It means that this particular key event is the last in a Unicode codepoint sequence.

The Android implementation is probably the weaker one as `KeyEvent::getUnicodeChar()` does not seem to support codepoint sequences of more than one key.

#### Quit request

I have added an `onQuitRequest()` input event that makes it possible to override a quit request coming from the system.

Think about pressing the window close button or using the contextual menu from the taskbar to perform the same action.
In those cases, the new event gets called and the user has a chance to save the game or to display a dialog window. The latter is what happens with SpookyGhost now.

![SpookyGhost Quit Confirmation](/images/SpookyGhost_quit_confirmation.png "SpookyGhost_quit_confirmation"){: .center-block :}


#### Post update callback

The new `onPostUpdate()` application event solves an old problem: being able to know the absolute transformation of a node before it gets rendered.

It gives a new meaning to query methods like `SceneNode::absPosition()` as it can now be called after all nodes have been transformed so you don't get stale data from the last frame.
I have also added a `SceneNode::setWorldMatrix()` method to further modify the transformation before rendering.

This change, together with a fix that caused an additional frame delay when rendering ImGui or Nuklear in presence of a scenegraph, made the overlay in `ncTiledViewer` perfectly synchronized with the movements of the underlying map.

![ncTiledViewer Imgui Overlay](/images/ncTiledViewer_ImGui_overlay.png "ncTiledViewer_ImGui_overlay"){: .center-block :}

### Automatic capacity extension for strings

String capacity is another quite annoying issue that I have been willing to address for a long time.
In the early days of nCine, I opted for fixed strings, you decided the capacity at construction time and the object allocated a buffer only once.

Great for performance, right? Possibly, but also a great source for bugs all around the place, with truncations happening when you least expect them.

I decided to change the default behavior: now strings reallocate if they need more space. You can keep appending text and the string object will make sure a truncation never happens. Of course, the user can still choose the old fixed behavior if needed.

This feature will make the _Savefile Size_ configuration entry in SpookyGhost obsolete. The program will not need to preallocate a big enough string to contain the project file.

### Fahien's suggestions and feedback
- I fixed an issue he discovered while working on `ncJump` in the behavior of `Array::setSize()`. Now it creates objects when extending the size, same as `StaticArray` and as you would expect. :man_facepalming:
- He requested the ability to move-construct and move-assign scenegraph nodes and to add a `clone()` method. This change will hopefully make both Rust and modern C++ users more comfortable. :house:
- To make it easier for `ncJump` to use the `docking` ImGui branch I exported the version tag of integrated software as CMake variables. It is now also possible to automatically download and extract a GitHub release instead of using a repository.
- Thanks to a smart suggestion I created a specialized version of the hash functions for strings that simplified a lot the declaration of hashmaps with a string key. They now work as expected with both string objects and array of characters.
- Additional feedback led to a new method to delete all children of a node and to a method to retrieve the current rectangle animation of an animated sprite.

### Minor Changes

- I have added the option to mark a color as the chroma key when loading an RGB texture. I can now load some Tiled example tilesets that use magenta for transparency.
- Another functionality dictated by Tiled capabilities is the custom frame durations for rectangle animations: every animation frame can have a unique duration different from others.
- With the support of ImGui v1.80 comes the new [Tables API](https://github.com/ocornut/imgui/issues/3740) which far surpasses the old Columns API.
- No more documentation is distributed with the nCine. It is now pushed online at every commit by GitHub Actions.
- For SpookyGhost, I needed separate blending functions for RGB and alpha channels so I now store additional states in the nCine wrapper class for OpenGL blending.
- I exposed yet another window property from the three desktop backends: its position. The user can now move the window around with code or query its current position.

I hope the article was worth your wait and you enjoyed the new features. :wink:
