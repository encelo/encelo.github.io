---
layout: post
title: nCine Dev Update 3
subtitle: Updates from November 2017 to February 2018
tags: [nCine]
---

A lot of work has been carried out during the last three and a half months.

First of all I have added some macros to allow asserts in the code, think about checks like this:  
`ASSERT_MSG_X(index < size_, "Index %u is out of bounds (size: %u)", index, size_);`  
The macro also invokes a breakpoint if a debugger is connected, allowing to inspect the context around the failure.
It was a long needed feature that I finally had the time to implement. :smile:

I have also found the time to configure [Uncrustify](http://uncrustify.sourceforge.net/) so that I could use it alongside [Artistic Style](http://astyle.sourceforge.net/) in order to further check consistency with my code conventions.

But the big change is the support of many features of C++11, a leap forward for the whole codebase!

I was wise enough to stop for a moment and think about the plethora of changes that the template library would require.
It is then that I realized that I would need to be disciplined and write unit tests first. I adopted [Google Test](https://github.com/google/googletest), which is well supported by [Qt Creator](https://www.qt.io/qt-features-libraries-apis-tools-and-ide/), and I started to write tests, plenty of them. The compiler helped with _code coverage_ data and [Gcovr](http://gcovr.com/) with HTML reports. After a month of work I had more than 500 tests and discovered many subtle bugs. :sweat_smile:

Now I was ready to work on supporting C++11.  
I setup a feature branch and started small by collecting low-hanging fruits first.
The very first change has been the replacement of all `NULL` occurrencies with `nullptr`, a thing that alone needed `set(CMAKE_CXX_STANDARD 11)`. :smile:  
Then more simple changes came, using `=delete` for special member functions that were previously `private`, the `override` specifier, delegating construtors instead of init functions, enum classes, alias declarations instead of `typedef`s and then I was ready for some bigger changes...

I moved the template library in its own namespace, `nctl`, standing for _nCine Template Library_. :wink:
I added a reverse iterator adapter for templated iterators, and added a sentinel to the hashmap iterator and to the list class in order to enable an easy and flexible way to iterate in both directions. But the code couldn't be modern without some range-based for loops and some lambda functions all around. :wink:

```cpp
nctl::Array<int> array;
for (int i : array_)
	printf("%d ", i);
```

```cpp
forEach(players_.begin(), players_.end(), [](IAudioPlayer *player){ player->pause(); });
```


Then it was time for the biggest features, move semantics and smart pointers.
In order to support the first one I have added _move constructors_ and _move assignment operators_ to all the containers, but also new insertion methods to allow the insertion of movable only objects, for example:
```cpp
inline void pushBack(const T &element) { operator[](size_) = element; }
inline void pushBack(T &&element) { operator[](size_) = nctl::move(element); }
```

Time for smart pointers, enter `nctl::UniquePtr` and `nctl::SharedPtr`!  
For the shared one to work reliably with multiple threads I have added atomics on all supported platforms.
I have replaced most of the raw pointers with unique ones and all factories now return a unique pointer to clearly show ownership transfer.
```cpp
nctl::UniquePtr<ITextureLoader> texLoader = ITextureLoader::createFromFile(filename);
load(*texLoader.get());
```

It has been a very long but also very satisfactory task and I'm proud of how the new codebase has turned out.
Now it's time again to think about something entirely new and exciting!
