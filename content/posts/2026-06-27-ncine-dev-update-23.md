---
date: "2026-06-27"
title: nCine Dev Update 23
description: Updates from September 2025 to June 2026
categories: [ Dev Update ]
tags: [ nCine, Rendering, OpenGL, Shaders, Benchmarks, nCTL, Windows, Filesystem ]
series: [ Dev Update ]
series_order: 23
---

Welcome back to another development update for the nCine, covering what has been accomplished in the last quarter of 2025 and the first half of 2026.

Before diving into the technical part of the article, I should probably mention that a few days ago marked the 15th anniversary of the [first commit](https://github.com/nCine/nCine/commit/6bf318de) of the project. :birthday:

Let's also cheer for the [new Hugo site](https://ncine.github.io/news/2025-09-28-new-site/) and the availability of [signed commits](https://ncine.github.io/news/2025-11-05-signed-commits/) :key:, two important infrastructure updates from last year.

## GrAIL

Probably the biggest development news is the work in progress toward a Render Hardware Interface (RHI) that will allow modern graphics APIs to be employed.
In the nCine the RHI is called _Graphics API Integration Layer_, or GrAIL.

The first iteration only supports Vulkan, but implements the underlying architecture that will make it possible to support other backends.
From the [HypeHype](https://advances.realtimerendering.com/s2023/AaltonenHypeHypeAdvances2023.pdf) rendering architecture presentation by Sebastian Aaltonen, I borrowed the concept of using an opaque handle (made of an index and a generation, like in my job system) for all the common resources, like textures, pipelines, and buffers. I also copied the concept of immutable bind groups, an abstraction over descriptors, from WebGPU.

The entry point of GrAIL, similarly to other modern APIs, is the device object. The device is not a traditional C++ base class, but a common header with different implementation files chosen at compile time by CMake. I chose to sacrifice run-time API selection to remove a layer of virtual call dispatch.

The plan is to use Vulkan on Linux and Windows (maybe I will add D3D12 in the future), to add WebGPU for Emscripten, and to test KosmicKrisp and MoltenVK on macOS before eventually adding a Metal backend.

Another important missing piece is the actual link between GrAIL and the scenegraph system, which will be handled by a new renderer class. The renderer will make it possible to remove rendering code from the nodes and will represent a chance to reorganize rendering data in a more data-oriented fashion.
For the time being I will also keep the legacy OpenGL code around, which means I will need to port the old system to this new renderer approach.

At the moment, to isolate the work on this feature, I'm taking advantage of the new `NCINE_WITH_SCENEGRAPH` CMake variable that allows the nCine to be compiled without any scenegraph-related classes, like the nodes, the render commands, or viewports.

There is still a lot of work before you can use GrAIL instead of OpenGL, but you can already try the `grail` branch on GitHub and play with the particle simulation test.

![Current GrAIL apptest, with compute simulated particles and a rotating nCine logo](/images/grlapptest_compute_texture.png "Current GrAIL apptest, with compute simulated particles and a rotating nCine logo")

## Batching with instances

While GrAIL represents the future, it does not mean that the OpenGL path would be deprecated any time soon. As a matter of fact, in June I have worked on optimizing regular sprite blitting.

As you might remember, the `RenderBatcher` class is in charge of automatically creating draw batches for every type of rendering commands that it encounters while navigating the sorted queues.

I have added another command collection path that works with regular sprites (so no `TextNode` or `MeshSprite` nodes) and uses a completely different approach.
Instead of using a Uniform Buffer Object (UBO) to gather the information about each sprite and expand their 4 vertices triangle strip into two triangles and 6 vertices, it uses instancing and vertex attributes.

This introduces two nice properties: with instances we can still use the 4 vertices triangle strip primitives for each sprite, and with the attributes we can gather the data in a Vertex Buffer Object (VBO). VBOs don't have the size restriction that UBOs have (usually 64 kb), meaning that it is now possible to batch a lot more sprites together in the same draw call.

The trick that enables vertex attributes to be used per instance and not per vertex is a call to `glVertexAttribDivisor()` with a divisor value of 1.

There is also another optimization that allows for bigger batches, and it also works when using UBOs: packing the data needed to render nodes.

For example, let's have a look at the `InstanceBlock` structure in the `sprite_vs.glsl` shader:

{{< tabs >}}

{{< tab label="Before" >}}

```glsl
// 104 bytes, rounded to 112 for alignment purposes
layout (std140) uniform InstanceBlock
{
	mat4 modelMatrix;
	vec4 color;
	vec4 texRect;
	vec2 spriteSize;
};
```

{{< /tab >}}

{{< tab label="After" >}}

```glsl
// 48 bytes in total, no padding needed
layout (std140) uniform InstanceBlock
{
	vec4 transform;
	vec4 translation;
	uint color;
	uint spriteSize;
	uint uvEndpointsU;
	uint uvEndpointsV;
};
```

{{< /tab >}}

{{< /tabs >}}

The new structure needs less than half the memory, allowing for more than double the number of sprites in the same batch.

On the C++ code side, the framework will try to first update the new uniforms, but it will fall back to the old fields if the process fails.
On the shader side, new functions will unpack the data into floats, trading the space gains for some additional low-effort per vertex computation.

### Benchmarks

I have tested the instancing on my slowest laptop, a Xiaomi Mi Notebook Pro with an i7-8550U, and further limited the CPU and GPU frequencies to the minimum to stabilize the results and to highlight any performance benefit on slow devices. I ran the updated `apptest_bunnymark` with 5000 bunnies and no V-Sync, then I toggled instancing on and off.

{{< mermaid >}}
xychart
title "Five seconds average runs"
x-axis ["Run 1", "Run 2", "Run 3", "Run 4", "Run 5"]
y-axis "Frametime (ms)" 0 --> 35
bar "A" [26.201, 27.067, 25.969, 26.232, 26.187]
bar "B" [19.890, 19.366, 18.398, 19.654, 19.780]
{{< /mermaid >}}

As you can see from the chart, enabling instancing cut the frametime to 74% of the baseline, from an average of 26.3312 ms to 19.4176 ms.

## Faster directory traversing on Windows

I am certainly not an expert in Windows API, but I remember reading an article about how to make directory traversal faster by using `FindFirstFileExA()` and its additional parameters, like [FIND_FIRST_EX_LARGE_FETCH](https://blog.s-schoener.com/2024-06-09-find-first-large-fetch/).
I then went to have a look in the `FileSystem` namespace and I experimented with the new approach. I removed some unneeded string copies, and I changed the API call from a simple `FindFirstFile()` to:

```cpp
hFindFile_ = FindFirstFileExA(buffer, FindExInfoBasic, &findFileData, FindExSearchNameMatch, nullptr, FIND_FIRST_EX_LARGE_FETCH);
```

{{< mermaid >}}
xychart
title "Scanning C:/Windows/System32 (4786 files)"
x-axis ["Run 1", "Run 2", "Run 3", "Run 4", "Run 5"]
y-axis "Time (ms)" 0 --> 4
bar [3.4946, 2.8509, 2.9092, 2.8022, 3.4254]
bar [3.2654, 2.3800, 2.3263, 2.3020, 2.4079]
{{< /mermaid >}}

In this first case, using the new API call lowered the average from 3.09646 ms to 2.53632 ms, or a new time that is 82% of the old one.

{{< mermaid >}}
xychart
title "Scanning C:/msys64/usr/bin (604 files)"
x-axis ["Run 1", "Run 2", "Run 3", "Run 4", "Run 5"]
y-axis "Time (ms)" 0 --> 1
bar [0.7142, 0.5205, 0.5406, 0.6141, 0.6587]
bar [0.5092, 0.5084, 0.5168, 0.4214, 0.4306]
{{< /mermaid >}}

In the second case, the new API call reduced the average from 0.60962 ms to 0.47728 ms, or a new time that is 78% of the old one.

## nCTL updates

### Optional class

While working on GrAIL, I needed a way to control when to construct objects that are fields of other objects. I usually do this with `nctl::UniquePtr`, by choosing the right time to call `nctl::makeUnique()`, but in this case I wanted to reserve the space for the object beforehand, and avoid allocating memory.

I ended up replicating some functionalities from `std::optional`, a class that is usually used to return an object or a value that can be interpreted as _null_ (when the optional class is not "engaged") when a function returns.

The class is really just a buffer as big as its template argument, and objects are created there with a placement new call. These features satisfy my requirements and that's how `nctl::Optional` was born, of course with its accompanying set of unit tests, as is tradition in nCTL.

### Iterators refactoring

All iterators have been refactored so that they obey the same invariants, and behave similarly to STL.

For example, a reverse iterator at the beginning of a container is now always constructed from a regular iterator at the end of the same container.
The same is valid for the symmetric situation, a reverse iterator at the end of a container.

```cpp
	/// Returns a reverse iterator to the beginning
	inline ReverseIterator rBegin() { return ReverseIterator(end()); }
	/// Returns a reverse iterator to the end
	inline ReverseIterator rEnd() { return ReverseIterator(begin()); }
```

I now explicitly check for those equivalences in the unit tests, together with other invariant checks, like checking that when I increment then decrement a `begin()` iterator, or do the opposite with an `end()` one, I don't alter it in a way that makes it no longer equivalent to `begin()` or `end()` themselves.

Last but not least, I rewrote the `operator!=` in all iterators to be consistent and just always negate the result from `operator==`.

### Pair class

Just like `nctl::Optional`, the template library keeps extending with some less used but sometimes useful classes, for example a reimplementation of `std::pair`.

It does not replace the custom pair implementation used by `nctl::UniquePtr` to store its deleter, as it is a class intended for general use cases.

While the optional class needed some new type traits to remove both the `const` and the `volatile` qualifiers from types, the pair class needs type decay when creating objects with `makePair()`.

### Hash containers refactoring

Hash containers have also been refactored. Hashmaps, for example, now accept a `nctl::Pair` when inserting a key and a value.

Most importantly, I have refactored the `HashMap`/`StaticHashMap` and `HashSet`/`StaticHashSet` classes so they don't duplicate any code when traversing the data structure.

The results of probing into them are now grouped in a structure:

```cpp
	/// The returning structure after probing for a key
	struct ProbeResult
	{
		/// The ideal bucket index for the hash
		unsigned int ideal;
		/// Only valid if the found flag is true
		unsigned int found;
		/// First empty node in a chain
		unsigned int empty;
		/// Previous node in a chain (for delta patching)
		unsigned int prev;
		/// True if the node contains a value
		bool foundFlag;
	};
```

For `HashMapList` and `HashSetList` classes, the total number of elements stored is now saved in a variable, instead of calculated from the buckets.

### Initializer list support

I have added yet another feature that makes nCTL containers more powerful and compatible with STL ones, the ability to use initializer lists to specify the initial content at construct time.

Take a look at this snippet from a `nctl::Array` unit test, for example:

```cpp
nctl::Array<int> newArray({ 0, 1, 2, 3, 4 }, Capacity);
```

The array is constructed with a copy of the objects passed in the initializer list, similarly to what you would do with a `std::vector`.

For this to work I had to add `#include <initializer_list>`, as there is no way to reimplement this header, the implementation is too tightly integrated with the compiler. But it does not bring in any STL dependency, the include is minimal and only serves the purpose of supporting the initialization through the curly braces.

Initializer lists can now be used with nearly all containers (`Array`, `HashMap`, `HashSet`, `List`, `SparseSet`) and unit tests have been updated to use them.

### Other additions

- I removed the fixed capacity option from the `nctl::String` class, and extended the size for the small buffer from 16 to 24 bytes.
- I have added a new `nctl::StringView` class that doesn't own memory and allows you to format C-style arrays of characters, similarly to `std::string_view`.
- The new `nctl::Span` class does something similar to `nctl::StringView`, but for arrays that are not characters.
- Surprisingly I wasn't checking for self-assignment in `StaticHashMap` and `StaticHashSet` classes, this has been fixed.
- I rewrote the functions in the `nctl::PointerMath` namespace, now all the alignment parameters have been extended from `uint8_t` to `size_t`.

## Hashing functions

This is another change that was needed for GrAIL, where I heavily use hashing for caching purposes.

The `fasthash64()` function now works more reliably, as it does not require padding to 64 bits anymore. I have also made it the default function for hashing keys in all `HashMap` and `HashSet` containers. It is a lot faster than _FNV1a_ and statistically stronger.

To reach this conclusion I added a micro-benchmark and some unit tests that compare the various hashing functions on different inputs.

I have also added a couple of functions that, given a 64-bit hash, can create a proper 32-bit version of it without losing too much information. This is a task that is often needed in GrAIL, where 64-bit Vulkan opaque handles are converted to 32-bit keys for containers.

### Benchmarks

Hashing a short string repeatedly with the various functions returns the following results on my Asus laptop in Performance mode.

{{< mermaid >}}
xychart
title "Hashing a 256 bytes string"
x-axis ["Sax", "Jenkins", "FNV1a", "FastHash64"]
y-axis "Bandwidth (GiB/s)" 0 --> 10
bar [1.08797, 0.871923, 1.11025, 7.83879]
{{< /mermaid >}}

The FastHash function is nearly 8x faster than all the others, achieving a bandwidth close to 8 GiB/s on a single core. :rocket:

## Application configuration structures

Yet another change that spawned from the work done on GrAIL. This was made to ease future extensions of the `AppConfiguration` settings when GrAIL will be merged.

Instead of having all the flags and values together as fields of the class, they are now organized in structures.
There is a `Logging` structure, a `Window` one, a `Graphics` structure, an `Audio` one, and so on.

{{< tabs >}}

{{< tab label="Before" >}}

```cpp
void MyEventHandler::onPreInit(nc::AppConfiguration &config)
{
	config.consoleLogLevel = nc::ILogger::LogLevel::OFF;
	config.resolution.set(1920, 1080);
	config.deferShaderQueries = false;
}
```

{{< /tab >}}

{{< tab label="After" >}}

```cpp
void MyEventHandler::onPreInit(nc::AppConfiguration &config)
{
	config.logging.consoleLevel = nc::ILogger::LogLevel::OFF;
	config.window.resolution.set(1920, 1080);
	config.graphics.opengl.deferShaderQueries = false;
}
```

{{< /tab >}}

{{< /tabs >}}

This gives a lot of room for future changes, and it has also been ported to the [environment variables](https://github.com/nCine/nCine/wiki/AppCfg-EnvVars) that you can specify when running an nCine executable.

## Minor changes

- The `Vector4f` and `Matrix4x4f` classes now align at 16 bytes, to help the compiler's auto-vectorization.
- On desktop, it is now possible to set the resizable flag of the window at run-time. The feature was already available in all backends and there was no reason not to expose it.
- By using `RUNPATH` for Linux executables and by copying MinGW dependency libraries when creating a package, those two platforms should now correctly support completely portable installations, without relying on system-wide installed libraries.
- GitHub Actions workflows now write a manifest JSON file with information about the packages they assemble. This in turn is used by the website to always offer the latest binaries for download.
- Texture loader classes are now just image loaders and are not coupled with OpenGL at all, facilitating their use with GrAIL.
- I have split some long CMake scripts in smaller chunks based on the platform, and moved them in subdirectories.
- Now that [Lua 5.5](https://www.lua.org/versions.html#5.5) is out, I have added proper support for it.
- For consistency I renamed all occurrences of _FullScreen_ to _Fullscreen_, _fullScreen_ to _fullscreen_, and _full_screen_ to _fullscreen_ in code and documentation, treating it as a single word.
- I fixed some JNI version checks before calling newer Android APIs, and now nCine runs again on my 2016 Xiaomi Mi 5 running Android 8! :robot:
