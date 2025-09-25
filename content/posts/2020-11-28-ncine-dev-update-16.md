---
date: "2020-11-28"
title: nCine Dev Update 16
description: Updates from the second half of May to the first half of November 2020
categories: [ Dev Update ]
tags: [ nCine, C.I., UTF-8, Android, Tools ]
series: [ Dev Update ]
series_order: 16
---

If you follow the project on [GitHub](https://github.com/nCine) you might have noticed a big development slowdown during the summer. I blame it on a combination of excessive heat and fatigue that led to a general lack of motivation and perseverance. :sunny:

Fortunately, this does not mean that development didn't resume at its normal pace or that there are no things to talk about in this article. :wink:

### Loading resources from memory

Let's start with a feature that was requested by a user on [Discord](https://discord.gg/495ab6Y): the ability to load a resource from a memory buffer.

To minimize changes and reuse interfaces I just implemented a `MemoryFile` class. You can open, close, seek, read and write to it just as if it were a normal file on disk.

This way a texture, a font, an audio buffer... everything can be loaded from memory, giving much more flexibility to the user.

### Fault-tolerant loading and reloading of resources

But loading from memory was not enough, I had more ideas to enhance the resource loading API and grant even more flexibility.
One of the main limitations of the system was apparent if the user tried to load a non-existent file: the whole application would just suddenly exit.

This behavior might be acceptable if you are debugging a game but it is incompatible with any kind of tool workflow, where the user might erroneously try to load a sound file in place of a texture.

The first idea I had to solve this was to provide additional classes that could attempt to load a specific data type and, if successful, be consumed by the corresponding resource.

Of course, those new classes alone would have not been enough, I also needed to remove all `exit()` calls from loader classes that take care of specific file format, like Png or Ogg Vorbis.

The new system was working as expected: should the user construct a `Texture` from a non-existent file the engine would straight out exit just as before. But now it was also possible to explicitly check if the loading process was successful by using one of the new data classes and then pass it to the resource object. In this example, the user would use a `TextureData` instance to load an image from the disk, check if the loading was successful, and then pass it to the `Texture` constructor.

While this solution was working sufficiently well I didn't like the fact that the user had to rely on those new classes to perform a safe loading and I scraped them altogether.

I took one step forward and made all resources class completely dynamic when it comes to loading data. Take again the `Texture` class, for example, you can now create an object with a default constructor, which might come useful when you need a pool of textures.
But what is even more interesting is that you can load data in a texture (or an audio buffer, or a font) multiple times. You can assign an empty texture to a sprite and it will initially render nothing. Later on, you can load an image and have it applied to your sprite, and even load a second one later on.

Anytime there is a loading error nothing will happen to the application nor to the texture object: it will retain whatever data had before the last attempt. I was inspired by the SFML [Texture](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Texture.php) class, with its default constructor and multiple load methods. In the future, I might even push further and have a [Sprite](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Sprite.php) that can be created with a default constructor and have a texture assigned at a later time.

I have also added a brand new feature: it is now possible to load uncompressed texels in a texture and PCM samples in an audio buffer, allowing for CPU generated procedural data. :tada:

### GitHub Actions

As I have written in a [news](https://ncine.github.io/2020-11-04-github-actions/) on the nCine site I now use GitHub Actions instead of Azure Pipelines for _continuous integration_.

I have been following the development of [Actions](https://github.com/features/actions) for some time and I think it has now all the features I needed so I spent some days converting the building scripts and testing the new _workflows_, as they are now called.

The nice thing is the integration: the code and the C.I. are very close together, on the same page. Besides, the configuration is easier as I don't have to visit a different site for setting the C.I. up.

![nCine page for GitHub Actions](/images/GitHub_Actions.png "nCine page for GitHub Actions")

As I was taking advantage of an advanced feature of GitHub I thought I could honor an old note I wrote a long time ago: make use of its [project management](https://github.com/features/project-management/) features by bringing my Trello tickets to Issues and having a roadmap with Projects.

There are no milestones or project boards yet but I started tracking some tasks with [issues](https://github.com/nCine/nCine/issues). :spiral_notepad:

I have also decided to simplify the development process by getting rid of the `develop` branch and just rely on feature branches, like in the [GitHub flow](https://guides.github.com/introduction/flow/) model.

### Decoding UTF-8 strings

While [Jugilus](https://github.com/Jugilus) was working on the [Gui Demo](https://jugilus.github.io/Jugimap-GuiDemo/JugimapGuiDemo.html) he asked me about supporting UTF-8 strings. At the time I had very little knowledge about the topic except for [the absolute minimum every developer must know](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/).

As many times with the nCine, my coding from scratch efforts is an excuse to research more about something, and the same happened this time. In a short amount of time, I knew a lot more about Unicode, code points, [decoding](https://www.instructables.com/Programming--how-to-detect-and-read-UTF-8-charact/) UTF-8, [testing](https://www.cl.cam.ac.uk/~mgk25/ucs/examples/UTF-8-test.txt) it, or how to interpret some of the information from a table like [this](https://www.compart.com/en/unicode/U+00e8).

![`apptest_font` rendering Unicode glyphs](/images/apptest_font_unicode.png "`apptest_font` rendering Unicode glyphs")

### Android soft keyboard

I always wanted to enable the use of the Android virtual keyboard to allow users to enter text in as it is expected from a mobile device, that is without plugging in a physical keyboard. :sweat_smile:

I just didn't have the will to dig into the Android API and JNI once more to support the feature. In the end, I was able to gather my strength and implement it.

One downside is that [toggling](https://developer.android.com/reference/android/view/inputmethod/InputMethodManager#toggleSoftInput(int,%20int)) visibility seems to be more reliable than querying and settings the visibility state, as per this StackOverflow question about [`showSoftInput` vs `toggleSoftInput`](https://stackoverflow.com/questions/13694995/android-softkeyboard-showsoftinput-vs-togglesoftinput). :disappointed:

### Tracy multiple memory pools tracking

I remember back at the end of March when I asked Tracy author on Discord if he was considering adding a way to tag allocations so that it would be possible to tell which custom allocator was responsible for it.

I didn't even have custom allocators at the time, but I was starting working on them. And now, after many months, we don't only have custom allocators in the nCine, but string tags in Tracy [v0.7.3](https://github.com/wolfpld/tracy/releases/tag/v0.7.3) allocation macros! :muscle:

![Multiple Memory Pools in Tracy v0.7.3](/images/Tracy073_memory_pools.png "Multiple Memory Pools in Tracy v0.7.3")

### Minor changes

- After a very long time since the last release, [Lua 5.4](https://www.lua.org/versions.html#5.4) is finally out. It broke a couple of things, from CMake older than version [3.18](https://blog.kitware.com/cmake-3-18-0-rc4-is-ready-for-testing/) not being able to find it to a compilation error due to a [missing constant](https://www.lua.org/manual/5.4/manual.html#8.3).
- The FileSystem API was updated to support Android assets: it's now possible to perform queries on asset files and traverse asset directories.
- A user made me notice that the root node was not being transformed, breaking the assumption that parent nodes affect children's transformations. I fixed it by slightly changing the nodes tree traversal.

Hopefully, this update was enjoyable enough that you are coming back for the next one. Stay tuned! :wink:
