---
layout: post
title: nCine Dev Update 4
subtitle: Updates from March to June 2018
tags: [nCine]
---

Plenty of work has been done during these few last months as I decided once again to update the engine renderer.
This time it went from using OpenGL 2 and OpenGL ES 2 to 3.3 and 3.0 respectively.

I started by rewriting all the shaders to support the new `in` and `out` keywords in place of the old `attribute` and `varying` ones.
For the sprite shader I also changed it in a way that it doesn't need any vertex data anymore, it generates a unit square by itself. 
I also took the chance to add support for [Uniform Buffer Objects](https://www.khronos.org/opengl/wiki/Uniform_Buffer_Object), one of the important new features of this OpenGL version.

As I wanted to share an UBO between multiple rendering entities I have written an OpenGL buffer manager so that a rendering command, say for example a command used to render a sprite,
can ask for a certain amount of bytes without worrying about alignment or creating a new buffer when there is no more free space left.
The manager is also responsible for mapping and unmapping memory once per frame.

For mapping I usually use a combination of `GL_MAP_INVALIDATE_BUFFER_BIT` and `GL_MAP_FLUSH_EXPLICIT_BIT`, as suggested on
the [Buffer Object Streaming](https://www.khronos.org/opengl/wiki/Buffer_Object_Streaming) wiki page, in order to maximize performances.
I don't use persistent mapping as the [ARB_buffer_storage](https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_buffer_storage.txt) extension is only available with newer versions of the API.
I have extended the manager to also handle VBO memory and thus enabling the use of a single common buffer for all the vertices that don't live in a custom VBO (which is a VBO that is only used by a single render command).

Another important addition is the support for the [KHR_debug](https://www.khronos.org/registry/OpenGL/extensions/KHR/KHR_debug.txt) extension in the form of debug groups and object labels.
This feature is really important with tools like [apitrace](http://apitrace.github.io/) and [RenderDoc](https://renderdoc.org/) that offer support for them.

![apitrace](/images/apitrace.png "apitrace")

Handling vertex formats has been renewed as well and there is now a pool of [Vertex Array Objects](https://www.khronos.org/opengl/wiki/Vertex_Specification#Vertex_Array_Object) that minimize the number of calls when changing formats.

One feature I was willing to add for a long time is the ability to render sprites with a custom mesh. It can be used to split the transparent outline from the opaque part of a sprite,
like it is described in this [article](https://community.arm.com/graphics/b/blog/posts/mali-performance-7-accelerating-2d-rendering-using-opengl-es), or to trim a particle down to the area with non completely transparent pixels,
or to animate with shape deformations. It also allows for the definition of a set of indices in order to reuse vertices, a feature that was not available in the engine before and that required some effort and testing.
As per vertex data, indices are handled by the memory manager and collected each frame in a set of common buffer objects.

The new OpenGL ES 3.0 Android renderer allows the use of the [ETC2](https://en.wikipedia.org/wiki/Ericsson_Texture_Compression#ETC2_and_EAC) compressed format and immutable textures.
For this second feature to work on desktop I need to check the presence of the [ARB_texture_storage](https://www.khronos.org/registry/OpenGL/extensions/ARB/ARB_texture_storage.txt) estension as OpenGL 3.3 doesn't offer it out of the box.  
Viceversa there is one thing I use on desktop OpenGL that is not supported by OpenGL ES 3.0: [glDrawElementsBaseVertex](https://www.khronos.org/registry/OpenGL-Refpages/es3/html/glDrawElementsBaseVertex.xhtml).
On Android I have to simulate this drawing command variation by using an additional offset.

For sure one of the most complicate new feature, as it has many corner cases that needed extensive debugging, is the automatic batcher.  
It allows to reduce the total number of draw calls issued by aggregating uniforms, vertices and indices from multiple commands into a single one. It can batch text nodes, regular sprites and custom mesh sprites.

For entities with external vertex data, like text nodes and custom meshes, it has to supply an additional vertex attribute to act as a mesh id, while for custom meshes it has to add artificial indices to sprites that do not use them if they are going to be batched together with sprites that have user supplied ones.

It's degenerate vertices and patched indices all the way down. :wink:

![RenderDoc](/images/RenderDoc.png "RenderDoc")

Some of those new features have a runtime setting, like switching automatic batching on and off:

```cpp
/// True if batching is enabled
bool batchingEnabled;
/// True if using indices for vertex batching
bool batchingWithIndices;
/// True if node culling is enabled
bool cullingEnabled;
/// Minimum size for a batch to be collected
unsigned int minBatchSize;
/// Maximum size for a batch before a forced split
unsigned int maxBatchSize;
```
Some others have an initialization configuration, like the dimension of each common VBO or the size of the VAO pool:

```cpp
/// Sets the maximum size in bytes for each VBO collecting geometry data
void setVboSize(unsigned long vboSize);
/// Sets the maximum size in bytes for each IBO collecting index data
void setIboSize(unsigned long iboSize);
/// Sets the maximum size for the pool of VAOs
void setVaoPoolSize(unsigned int vaoPoolSize);
/// Enables OpenGL debug context
void enableGlDebug(bool shouldEnable);
```

There is also a bunch of new rendering statistics that are aggregated per frame and displayed on screen, like the amount of video memory used by textures and buffer objects, or the number of VAO pool recycles.

![apptest_sinescroller](/images/apptest_sinescroller.png "apptest_sinescroller")

Rendering should now be more efficient on all platforms, with fewer draw calls and more sprites on screen! :muscle:
