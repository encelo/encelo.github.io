---
layout: post
title: nCine Dev Update 15
subtitle: Updates from April to the first half of May 2020
tags: [nCine]
---

I have spent nearly two months on a big task this spring: custom memory allocators.
They can be useful in different scenarios to alleviate the performance cost of allocating and deallocating memory.

But before diving into that I had to be sure that the containers were ready.

### Split allocation and construction

Just like with STL ones, there has always been a difference between capacity and size in nCine containers.

Size represents the number of elements currently stored in the container and it has an initial value of zero.
The capacity defines the maximum number of elements that can be stored in a container.

Some containers like the `Array` class are dynamic, similarly to an STL `vector`, it can grow to accommodate more elements by allocating a bigger chunk of memory and copying over the old ones.
If you already know how many elements the container is going to store during its lifetime, you can reserve an initial capacity to avoid reallocations and copies.

Now, the big difference between an old nCine `Array` and an STL `vector` is in their construction.
The first will allocate its memory like this: `array_ = new T[capacity_]`.

What is wrong with a perfectly safe statement like that?
The problem is that the class is both allocating memory and constructing elements up to its maximum capacity.

This behavior has several implications:

- Time is spent constructing objects even if no one asked for that
- The `T` class needs a default constructor
- New elements can only be copy-assigned or move-assigned
- It is not possible to emplace new elements
- Popping elements does not destruct them

The solution is to split the allocation phase from the elements construction.
The `Array` allocation then becomes: `array_ = static_cast<T *>(::operator new(capacity_ * sizeof(T)))`.

When we want to add new elements they will be copy-constructed (`new (extendOne()) T(element)`) or move-constructed (`new (extendOne()) T(nctl::move(element))`) in the array using a [placement new](https://en.cppreference.com/w/cpp/language/new#Placement_new) operator. Something similar will happen when we emplace back (`new (extendOne()) T(nctl::forward<Args>(args)...)`) an element.
Popping elements will destruct them as you would expect from an STL `vector` class.

There is also an additional important optimization based on type traits: if an element is [trivially destructible](https://en.cppreference.com/w/cpp/types/is_destructible) then its destructor will not be called upon deallocation.

All nCine containers and the relative unit tests and benchmarks were modified and updated to support the new behaviors and functionalities.

### Custom memory allocators

Now that the containers have a clear and separated allocation phase it is time to work on the allocators themselves.

They are always initialized with a pointer to the beginning of a memory arena and its size. This makes the allocators very flexible as the memory can be allocated in different ways, it can be a region accessible by the CPU or mapped by the GPU. It can even be a subset of a region managed by another custom allocator!

The memory arena is usually allocated only once by the operating system then control is passed to custom allocators. They are generally faster to allocate and deallocate memory and they can achieve even more performance within the use cases they were designed for.

One of my main references has been an article on Gamedev.net called [C++: Custom memory allocation](https://www.gamedev.net/tutorials/_/technical/general-programming/c-custom-memory-allocation-r3010/). It shows a basic interface for allocators plus four different implementations.

The engine implements the same four types of allocators:

- The **Linear** allocator is the simplest one and allocates in constant time. It doesn’t have any kind of memory overhead beyond the alignment requirements but it doesn't do any allocation bookkeeping and thus it cannot deallocate. It supports a _clear_ operation to deallocate all memory at once though, a useful feature that can be used to implement a fast per-frame scratchpad memory.
- The **Stack** allocator introduces a header at the start of each allocation to keep track of the address adjustment made to satisfy the alignment requirements. This makes it possible to deallocate the last allocation made.
- The **Pool** allocator is a very classic allocator type for game development. It allows very fast allocations and out-of-order deallocations when the allocations have all the same size and the same alignment requirement.
- The **Free List** allocator is capable of serving allocations of different sizes and alignments and deallocating out-of-order.

There are also some differences between the article and my implementation:

- The allocator interface has no virtual methods but relies on function pointers.
- The allocator interface supports reallocation. It can happen in place or by allocating a bigger chunk and copying the old elements, depending on the situation. Reallocation is very important to properly support Lua garbage collector and it is also used by Nuklear memory functions.
- The Free List allocator performs fast compaction on deallocation to keep external fragmentation low. It has also three different fit strategies for allocations: _Best_, _Worst_, and _First_.

#### Allocation manager

The entry point for allocators setup is the `AllocManager` class. It is responsible for creating allocators before the first allocation is ever made and for providing those allocators to the application so that memory can be acquired and released.

Ensuring that the allocation manager initializes the allocators before even static variables have a chance to allocate memory is the tricky part.
The first solution I investigated was the [Nifty Counter](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Nifty_Counter), but it is a dirty hack that can have performance implications.

I then changed my implementation to use specific compiler features such as the [init_priority attribute](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Attributes.html#C_002b_002b-Attributes) of GCC and Clang, and the [init_seg pragma](https://docs.microsoft.com/en-us/cpp/preprocessor/init-seg?view=vs-2019) of MSVC.

There are a couple of optional defines in the `AllocManager` class that can be used to tweak some aspects.

The first is `OVERRIDE_NEW`, by defining it the class will redefine the global `new` and `delete` operators so that every allocation and deallocation will use custom allocators transparently.

The second one is `RECORD_ALLOCATIONS`, by defining it every allocator will record an entry for each allocation and deallocation made, with information about the number of bytes, the alignment, the timestamp, and so on.
You can then use some included functions to print all entries or to query the list and know which allocation has never been freed.

#### Apptest_allocators

Last but not least, a new application test has been added to show how allocators work: `apptest_allocators`. It uses some custom ImGui drawing code to display a memory map similar to a disk partition editor.

<div class="embed-responsive embed-responsive-16by9">
  <iframe width="640" height="360" src="https://www.youtube.com/embed/2kAoyVvgLyo" frameborder="0" allowfullscreen></iframe>{: .center-block :}
</div>
<br/>

### Minor Changes

* There is now a [node inspector](https://www.youtube.com/watch?v=lPMd8fI99gI) in the debug overlay. It is useful for understanding what's going on in the scene and for changing node properties.

* The FileSystem API has been extended to support Android assets files and directories.

I hope you found this article interesting and useful.
Ah, and don't forget to check the [2020.05](https://github.com/nCine/nCine/releases/tag/2020.05) release! :wink:
