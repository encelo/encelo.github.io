---
date: "2019-01-08"
title: nCine Dev Update 7
description: Updates from the second half of December 2018
categories: [ Dev Update ]
tags: [ nCine, nCTL, Benchmarks ]
series: [ Dev Update ]
series_order: 7
---

Just a few weeks after the last update here I'm again to write the next one in which I'm going to show you the performance of my new hash table implementation.

It is based on the [Leapfrog Probing](https://preshing.com/20160314/leapfrog-probing/) article by Jeff Preshing, where the author explains a new probing algorithm for collision resolution when using [open addressing](https://en.wikipedia.org/wiki/Open_addressing).

My version is not multi-threaded but in return it is able to remove elements, a task that turned out to be quite more challenging than what I expected in the beginning. :sweat_smile:
I also took the opportunity to change the default hash function to [FNV-1a](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function#FNV-1a_hash).

My previous implementation was based on [separate chaining with list head cells](https://en.wikipedia.org/wiki/Hash_table#Separate_chaining_with_list_head_cells), a pretty common technique that allows for unlimited number of stored elements in the face of many potential cache misses when the load factor increases.

The following results are a courtesy of the [Google Benchmark](https://github.com/google/benchmark) library, the latest third-party software integrated in the nCine.
They have been recorded on an [Intel i7-8550U](https://ark.intel.com/products/122589/Intel-Core-i7-8550U-Processor-8M-Cache-up-to-4-00-GHz-), running Arch Linux with the `performance` scaling governor and a fixed frequency of 800 MHz.

The tests have been performed on different implementations, they all have in common an `unsigned int` for the key and the value and 1024 buckets:

- `StaticHashMap` is the new open addressing hash map in a version that takes the number of buckets as a template argument, similarly to `std::array`
- `HashMap` is the new open addressing hash map in a more classic version that allocates on the heap
- `HashMapList` is the old hash map with separate chaining
- `unordered_map` is the standard hash table implementation from the STL

Some tests have a number after their name representing either the number of elements already in the hash table, like for _Copy_, _Retrieve_ or _Clear_, or the number of times the operation is repeated, like for _Insert_, _Remove_ or _ReverseRemove_.

|Benchmark         | StaticHashMap | HashMap  | HashMapList | unordered_map |
|------------------|--------------:|---------:|------------:|--------------:|
|Creation          |        443 ns | 34106 ns |    86589 ns |      13143 ns |
|Copy/256          |       2859 ns | 31501 ns |    78531 ns |    1656125 ns |
|Copy/512          |       2912 ns | 31004 ns |   745993 ns |    3600840 ns |
|Copy/768          |       2895 ns | 34312 ns |  1075128 ns |    4931709 ns |
|Insert/256        |       4725 ns |  9670 ns |     9313 ns |    1581495 ns |
|Insert/512        |       8423 ns | 19566 ns |   674695 ns |    3169524 ns |
|Insert/768        |      13896 ns | 33050 ns |  1007072 ns |    6504032 ns |
|Retrieve/256      |         66 ns |    67 ns |       75 ns |         72 ns |
|Retrieve/512      |         71 ns |    74 ns |       81 ns |         72 ns |
|Retrieve/768      |         76 ns |    77 ns |       83 ns |         73 ns |
|Clear/256         |       1733 ns | 22215 ns |    42527 ns |    1541229 ns |
|Clear/512         |       1733 ns | 21789 ns |   701556 ns |    3078237 ns |
|Clear/768         |       1734 ns | 22076 ns |  1025812 ns |    4609838 ns |
|Remove/256        |       5083 ns | 25895 ns |    35680 ns |    1565479 ns |
|Remove/512        |       9459 ns | 38793 ns |   703407 ns |    3136167 ns |
|Remove/768        |      14828 ns | 53607 ns |  1040247 ns |    4662149 ns |
|ReverseRemove/256 |       4906 ns | 25564 ns |    35849 ns |    1560456 ns |
|ReverseRemove/512 |       9620 ns | 38567 ns |   700163 ns |    3113046 ns |
|ReverseRemove/768 |      14527 ns | 51159 ns |  1031670 ns |    4669154 ns |

Last but not least, I have replaced all the occurrences of the classic `rand()` function with a new [PCG32](http://www.pcg-random.org/) random number generator based on the official [minimal C implementation](http://www.pcg-random.org/using-pcg-c-basic.html).

It has a higher statistical quality while also being a lot faster. In my microbenchmarks, with the same CPU settings as the hash map ones, it is able to generate 1024 integers in around 14203 ns, as opposed to 74303 ns when using simple `rand()` calls.
