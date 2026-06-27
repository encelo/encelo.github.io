---
date: "2026-03-27"
title: An Intro to Wave Intrinsics
description: Some concrete use cases using HLSL
tags: [ Rendering, Shaders ]
---

Most graphics programmers know about the concept of a warp (or a wave in AMD parlance).
It is a group of GPU threads (or lanes), typically 32, though some architectures use more, that execute the same instructions in lockstep.

Maybe not all of them know about wave intrinsics, though. They are a group of special GPU instructions exposed in HLSL (and similarly in other shading languages) that a shader programmer can use to orchestrate work across the threads in the same warp.

## Enforcing Uniform Control Flow

Let's start with a basic example, using the `WaveAllTrue(expression)` intrinsic.
This instruction returns `true` only if all threads evaluate the expression as `true`.

We said previously that all waves share the same instructions, but they operate on per-lane values of the same variable, stored in vector registers.
It's like running the same function with different parameters, the return value won't be the same.
Wave threads are similar: they have an execution context determined by a set of vector registers, and each thread sees its own version of the same variable.

That's why they can evaluate the same condition in different ways and why knowing whether all lanes produce a uniform result with `WaveAllTrue` can be so useful.

Let's suppose that we have a condition called `fastPath` that tells us if a shader computation can use a less general but faster approach.

In this case we could write a snippet like this:

```glsl
bool useFastPath = WaveAllTrue(fastPath);

if (useFastPath)
{
    // All threads go through the fast path
}
else
{
    // All threads go through the general path
}
```

What does this achieve? We have reduced divergence on the fast path.
Now either all lanes remain active for the fast path or none of them do.

Why is this beneficial? Why not just write `if (fastPath)`?

Because in this case the GPU evaluates `fastPath` per lane and uses an execution mask to track which lanes are active. This mask enables or disables lanes for subsequent instructions. Lanes where the condition evaluates to `true` remain active, while the others are masked off. The hardware then executes the `if` body with that mask, flips it, and executes the else path for the remaining lanes.

From the programmer's perspective this looks like a branch, but in practice both paths are typically executed sequentially under different masks.

> [!NOTE]
> The execution mask persists across control flow, and wave intrinsics operate over the currently active lanes rather than the full wave.

## Delegating Work to a Single Lane

Other times we want to spare most of the threads from doing an expensive operation, especially an atomic one.
An atomic operation would often be serialized between threads, so it makes sense to reduce its use.

We can do exactly this with an intrinsic like `WaveIsFirstLane()` and the help of another one called `WaveReadLaneFirst(expression)`.

Let's have a look at the following snippet first.

```glsl
uint oldValue;
if (WaveIsFirstLane())
{
    Buffer.InterlockedAdd(bufferOffset, addend, oldValue);
}
oldValue = WaveReadLaneFirst(oldValue);
```

In this example the expensive `InterlockAdd` operation is guarded by a wave intrinsic. This pattern ensures that only the first active lane in the execution mask will execute the `if` statement.
After performing the operation on the first thread, the value is broadcast to the remaining lanes by using the `WaveReadLaneFirst()` function.

## Reserving Space in a Buffer

The previous example can be further expanded to show how it is possible to reserve some space in a buffer for the whole warp to operate on it without stomping on other waves' space.

Suppose that each thread has a different number of items to write to a buffer.
How can we know how many items the whole warp needs to reserve? And how can we be sure that each thread writes its items in a sub-slice of the allocated space?
Yet again, wave intrinsics come to our aid.

```glsl
// The number of items that this lane has to write to the buffer
const uint laneNumItems = ...;

// The offset in the reserved space where this lane can write its items
const uint laneOffset = WavePrefixSum(laneNumItems);

// The starting offset for the space reserved by the whole warp
uint absoluteOffset;

// The following statement will be executed only by the last lane in the warp
if (WaveGetLaneIndex() == WaveGetLaneCount() - 1) 
{
    uint totalNumItems = laneOffset + laneNumItems;
    CounterBuffer.InterlockedAdd(0, totalNumItems , absoluteOffset);
}
absoluteOffset = WaveReadLaneAt(absoluteOffset, WaveGetLaneCount() - 1);

// Each lane writes its items at the designated offset
for (uint i = 0; i < laneNumItems; i++)
{
    DataBuffer[absoluteOffset + laneOffset + i] = value;
}
```

Let's analyze the code in more detail. First of all, each lane has its own number of items to store.
Then we encounter a new intrinsic: `WavePrefixSum()`. This instruction computes a prefix sum of `laneNumItems`. Each lane receives the sum of all previous lanes, excluding its own value. So if the first active lane wants to write 2 elements, the second 3, and the third 1, the prefix sum would be 0 at the first lane, 0+2 at the second, and 0+2+3 at the third.
This gives us the offset at which a lane needs to write its items.

The next `if` statement executes only on the last lane in the warp, using `WaveGetLaneIndex()` to get the index of the lane executing it and `WaveGetLaneCount()` to know the total number of lanes in the warp. Inside the statement, only the last lane will perform the atomic add to reserve space for the total amount of items to store.
To determine the size, it adds its number of items to its prefix sum.

Now every lane knows both the absolute and local offsets to correctly append their items to the buffer without affecting the others.

## Conclusions

We have seen different ways in which wave intrinsics can help orchestrate the work between the threads composing a warp.
Used wisely, they can improve performance in more complex patterns.

Nevertheless, always profile your baseline before implementing them in your code.
