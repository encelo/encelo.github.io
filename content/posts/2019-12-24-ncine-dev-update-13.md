---
date: "2019-12-24"
title: nCine Dev Update 13
description: Updates from the second half of September to the first half of December 2019
categories: [ Dev Update ]
tags: [ nCine, Rendering, Emscripten, Nuklear ]
series: [ Dev Update ]
series_order: 13
---

A lot of work has been put into the project as usual during those last months of the year.

Plenty of new and important features have been added to the engine, many of them are related to extending the capabilities of sprite rendering.

![apptest_anchor](/images/apptest_anchor.png "apptest_anchor")

### JugiMap

I have been recently contacted by [Jugilus](https://github.com/Jugilus), [JugiMap](http://jugimap.com/)'s author, for a collaboration. He was interested in the nCine as a sprite rendering backend, alongside [Cocos2d-x](https://www.cocos.com/en/), [AppGameKit](https://www.appgamekit.com/), and [SFML](https://www.sfml-dev.org/), to test the tool [integration API](https://github.com/Jugilus/JugiMapAPI) and preview in real-time exported maps and logic.

He made a list of requirements for features that were not in the engine at the time but that every other backend had.
To be honest the features he mentioned were, in a way or another, already in my to-do list so I just had to change my priorities to accommodate the need of the project. :wink:

The features I have implemented from his list are the following:

- **Custom anchor points**

  It allows a node to be anchored to a point different than its center, like one of the four corners, and be transformed relative to it.
  The feature needed a lot of refinements to work in every case and with every type of node.
- **Non-uniform scaling**

  Nodes can now be scaled independently along the horizontal or the vertical axis.
  This feature was already requested by a tester of ncParticleEditor a long time ago. :sweat_smile:
- **Texture flipping with a boolean flag**

  This change makes a lot of sense as you can now query and check if a sprite is flipped along one of the axes.
  Previously flipping was just an action to perform and there was no way to tell whether the sprite was flipped or not.
  This also means that, just like the custom anchor point, if you change something in the sprite, like the texture, the engine needs to automatically reapply any flipping.
- **Custom blending factors**

  Another feature requested by a tester of ncParticleEditor a long time ago. :smile:
  The user can now specify a custom source or destination factor for blending or use one of the presets, allowing for [premultiplied alpha](https://en.wikipedia.org/wiki/Alpha_compositing#Straight_versus_premultiplied) textures or additive effects.

I was very happy to have the API demo test as a big stress test for the sprite rendering, it highlighted serious issues like:

- Frame latencies with parent/child transformations
- Drawable nodes culling not working with negative scaling
- One frame delay to update a text node cached boundaries

You can find the project repositories on GitHub: [ncJugiMapAPIDemo](https://github.com/nCine/ncJugiMapAPIDemo) and [ncJugiMapAPIDemo-data](https://github.com/nCine/ncJugiMapAPIDemo-data).
You can also test the Emscripten [web test](https://ncine.github.io/ncjugimapapidemo/).

![ncJugiMapAPIDemo](/images/ncJugiMapAPIDemo.png "ncJugiMapAPIDemo")

### Emscripten enhancements

The Emscripten port has seen some improvements and fixes too. First of all, it can now build with version 1.39.0 or newer, which made the [LLVM WebAssembly backend](https://v8.dev/blog/emscripten-llvm-wasm) the [default](https://github.com/emscripten-core/emsdk/pull/373).

Another important fix is the correct handling of browser window resizing: an Emscripten application should now be correctly displayed regardless of browser window size or fullscreen state.
Continuous resizing should now also work for desktop native versions where the perspective matrix was previously not updated. :sweat_smile:

Deploying a nCine based tool to Emscripten makes a lot more sense as it is now possible to load and save files locally. It means you can load a file from your computer into a web build or save it from there to your computer. You can test this feature in the updated [ncParticleEditor](https://ncine.github.io/ncparticleeditor) web test. You will also notice the support for ImGui custom font loading in the form of [FontAwesome](https://fontawesome.com/) icons. :wink:

Last but not least, after a long tribulation, Emscripten builds can now use automatic render commands batching! :muscle:
There is just one small catch, the batch size is fixed. :weary:

Sure, you can configure it on initialization, but it is fixed, there is no minimum or maximum, the application will collect a certain amount of commands per batch. It will split a batch if it has reached the predetermined size or it will render single commands unbatched if there aren't enough.
You should also keep this number quite small to prevent the D3D shader compiler in ANGLE from taking a lot of time. :smile:

### Nuklear integration

[Nuklear](https://github.com/Immediate-Mode-UI/Nuklear) is an immediate UI similar in concept to [Dear ImGui](https://github.com/ocornut/imgui) but skinnable. I have added this integration as I expect a game to use Nuklear and customize the graphics while a tool would use the superior flexibility of Dear ImGui.

![Nuklear integration](/images/Nuklear_integration.png "Nuklear integration")

### Additional improvements

There are a lot of smaller things that have been added during this period:

- There are two new application events you can subscribe to, `onSuspend` and `onResume`. Frame and profile timers will not take into account the time while the application has been suspended.
- There is now a `ColorHdr` class that allows for unclamped floating-point color values which can be used for supporting HDR in tools or demos.
- Sorting of render commands is now stable, there should be no more random popping of a sprite in front of another. Two commands with the same material sort key will be sorted according to their creation time.
- The 4x4 matrix class supports in place transformations. It means that less memory will be used and fewer multiplications will be performed when translating, rotating or scaling in place.
- The deletion of children scene nodes upon parent destruction is now optional.
- As usual, the integrations with ImGui and Tracy have been updated to support the latest versions at the time of writing.

Enjoy the new nCine features and this festive season, whether you celebrate Christmas or not. :snowman:
