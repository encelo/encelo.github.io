---
date: "2022-09-07"
title: nCine Dev Update 19
description: Updates from the second half of January to the end of July 2022
categories: [ Dev Update ]
tags: [ nCine, Rendering, Shaders, AppTests, OpenGL, nCTL, Lua, Android ]
series: [ Dev Update ]
series_order: 19
---

Yet another update coming after a very long time since the previous, apologies for that. Well, at least it comes packed with a lot of enhancements from the last months. :muscle:

## Custom shaders

Probably the biggest feature of 2022, and the culmination of work that started with viewports a year ago, is the support for custom shaders.

You can now write your own vertex and fragment shaders and assign them to a node. All of that is possible while also retaining the usual automatic batching. :astonished:
In combination with viewports, it is now possible to create complex scenes, featuring post-processing and modern effects.

Take for example the new `apptest_shaders`, it shows regular and mesh sprites, all of which use custom shaders, custom vertex attributes, and they preserve automatic batching.
You will notice that some sprites are rendered with normal and specular mapping and that the light position can be moved in real-time! :bulb:

That's not all, with the ImGui interface or with the keyboard, you can enable a full screen gaussian blur post-processing or a bloom effect.
The latter is achieved with a quite complex viewports setup, with off-screen rendering, down-sampling, blurring, up-sampling, and compositing.

![apptest_shaders](/images/apptest_shaders.png "apptest_shaders")

But the road to achieving all that was quite long and required many months. It all started by checking out a very old branch in which I was planning out how to approach the matter in question.

### ShaderState class

In the first iteration the `ShaderState` class, the one handling shader uniforms, was embedded inside a `DrawableNode`.
The user could then set a lambda function through the `ShaderState` object that the parent node would call each time it was going to be rendered.

In the second one, I moved the `ShaderState` class outside, so that nodes using the default shaders would know nothing about it.
To change uniforms you could query the culling state of a node in the `onPostUpdate()` callback and update their values.

### OpenGL layer

The OpenGL classes have seen upgrades too, for example in the way shaders and shader programs were created and loaded. Shaders are now just another resource, they can be loaded from files or strings multiple times, and if they fail to compile this would only affect the nodes that use them for rendering.

Two other important upgrades were fundamental to support the normal mapping and bloom effects in `apptest_shaders`: the first is multi-texturing and the other is multiple render targets.
The former allows more than one texture to be used as input for a shader, while the latter allows a shader to write on more than one texture.

The `GLShaderAttributes` class has been deleted and its functionality has been moved inside the `GLShaderProgram` class. Missing the flexibility of changing the attribute setup per node is not going to affect the rendering at all, while the change simplifies the code of many classes with a possible uplift in performance.

If you run your game with a debug context inside a graphics debugger like RenderDoc, you should see additional information thanks to more `glObjectLabel()` calls.

### Viewports

Viewports have been upgraded as well. First of all, setting up a chain of viewports for rendering is a lot easier now as you can directly access the `Viewport::chain()` array instead of using multiple obscure and error-prone calls to `setNextViewport()`. It is also possible to easily set up cycles for multi-pass rendering techniques.

The second important change, one that simplifies the internals while adding flexibility, is the removal of the texture inside the `Viewport` class. A `Texture` object should now be passed to a viewport from outside, a solution that opens a range of possibilities, from sharing a texture between multiple viewports to using the output of one as the input of another.

Those two enhancements are at the base of the blur and bloom setups in `apptest_shaders`.

### More changes

There is now a new `onResizeWindow()` callback that the game can use to know when the window resolution changes. It comes in handy to recreate viewport textures if you have a post-processing setup in place.

The example Lua script has been updated as well to show how you can use viewports and shaders with scripts.

## Static string class

Like many other items on my to-do list, this was something I have been thinking about for some time. Having a string class that does not allocate space for characters should remove the need for a plain old `char` array while retaining all the useful formatting, appending, and comparing methods of the standard `String` class.

I have begun to use it in new code and tried to backport in some old cases where it made the most sense.
As usual, it comes with a battery of GTest unit tests that should ensure its correct functioning.

## Fixes to viewports

Apart from the big viewports API overhaul made in the `custom_shaders` branch, there were other small issues that I discovered after merging the corresponding branch.

For example, if multiple viewports were rendering the same scene node, its `update()` method would have been called multiple times. :sweat_smile:
This is now fixed by using a counter that stores the last frame a node has been updated by any viewport.

I have inverted the position and rotation values used by a camera so that it feels like an object that you can move in the scene.
It leaves back the old concept of simulating movement by moving the rest of the world in the opposite direction.

When using the Qt5 backend the viewports were not rendering correctly as Qt5 uses its own framebuffer object to render a `QOpenGLWidget` off-screen and composite it later.

## Rendering order based on node visit

This change brings the nCine closer to the behavior of other frameworks like [Godot](https://docs.godotengine.org/en/stable/tutorials/2d/canvas_layers.html#canvaslayers), [Defold](https://defold.com/manuals/gui/#draw-order), or [Cocos](https://docs.cocos.com/creator/manual/en/ui-system/components/engine/priority.html#ui-node-ordering).

The user can still affect the drawing order of nodes with layers, but by default, it is dictated by the visiting order of the node in the scenegraph. This change makes the order more reliable, intuitive, and similar to what users are already used to.

## Safer Lua pointers dereferencing

This is a very important change for scripters, before this change they might have encountered a crash too frequently while working with user data pointers.

Let's say you created a nCine object inside a Lua script, like a sprite. The engine would then create a `Sprite` object and a `UserDataWrapper` object.
The latter would be set to contain the pointer and type of the sprite, added to an array of wrappers, then its pointer returned to the user inside a light userdata.

You could have now used this variable to change the properties of the original sprite that you just created. But if you passed a non-valid wrapper, like one containing the pointer of a recently deleted object, the application would just crash.

This should not be the case anymore: every Lua function that operates on objects like sprites, textures, fonts, particle systems, and so on, will now check if the variable used is valid.
There is no wrapper object anymore, the light userdata contains the pointer to the native object and it is checked by accessing a hashmap where the key is the pointer itself and the value is its type.

No more arrays to iterate, if the hashmap does not return a value or if the type is not the expected one, the operation is not carried out and nothing happens!

## Updates to the Android building process

It has been quite some time before I had a look at the Android Developers documentation, the Android Gradle plugin, and the rest of the SDK tools.

The `build.gradle` script has been rewritten to support the latest plugin version. While doing so, all source files have been moved inside an `app` directory that now represents a separate building module.

A minor Android change is the support of the `description` attribute in the manifest.

## Minor changes

- The vector, quaternion, and matrix classes now initialize their values. A small change that will make happy some users without affecting performance.
- Thanks to the support for `glDebugMessageInsert()`, the OpenGL debug context should produce some new debugging messages.
- Hashmaps and hashsets could not perform a rehash if they contained non-copyable objects.
- The window resizable option has been fixed when using the Qt5 backend.
- The `Font` class can now use an external `Texture` object, enabling texture sharing among multiple nodes and modifications while the texture is in use.
- The color classes use a group of variables instead of a `StaticArray` for channels. It should help with understanding the color value in a debugger.
- Starting with macOS 10.12, `clock_gettime_nsec_np()` is the preferred way to query for a timer and makes the old `mach_absolute_time()` unsafe and deprecated.
- An audio player will now check if it's possible to register itself in the audio device before setting its state to `PLAYING`.
- The Lua API can now manipulate vectors with four elements.

I hope you enjoyed this development update installment and some of the background work around custom shaders.
