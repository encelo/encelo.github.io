---
date: "2025-09-22"
title: nCine Compilation Benchmark 3
description: How long does it take to compile the nCine in 2025?
categories: [ Compilation Benchmarks ]
tags: [ nCine, Build, Benchmarks ]
series: [ Compilation Benchmarks ]
series_order: 3
---

Compilation speed directly affects iteration time when developing an engine. Since nCine continues to grow, I thought it would be interesting to measure how long a full build takes today compared to 2018, when I wrote the [first article](/2018-03-04-ncine-compilation-benchmark), and 2022, when I wrote the [second one](/2022-10-06-ncine-compilation-benchmark-2).

I was thinking about writing this since the end of last year, when I bought both the second hand Mac and the small 14" Intel laptop.

Just like in the past, let's start with the hardware and the software I've used for my tests.

## Hardware and software

### Asus ROG Zephyrus G15 (2022)

#### Hardware

- AMD Ryzen 7 6800HS
- 16GB RAM, DDR5 4800 MHz, dual channel
- Western Digital SN735 NVMe SSD, 1TB
- NVIDIA GeForce RTX 3060, 6GB GDDR6

#### OS

- Microsoft Windows 11 Home (x64)
- Arch Linux (x86_64), Linux 6.16.8

### Alurin Flex Advance

#### Hardware

- Intel Core i5-1235U
- 16GB RAM, LPDDR4X 4266 MHz, single channel
- Western Digital Blue SN580 NVMe SSD, 512GB

#### OS

- Microsoft Windows 11 Professional (x64)
- Arch Linux (x86_64), Linux 6.16.8

### Mac Mini M1 (2020)

#### Hardware

- Apple Silicon M1
- 8GB RAM, LPDDR4X 4266 MHz, single channel
- NVMe SSD, 256GB

#### OS

- macOS Tahoe 26

### Compilers and tools

#### Arch Linux

- GCC 15.2.1
- CMake 4.1.1
- Ninja 1.12.1

#### macOS

- AppleClang 17
- CMake 4.1.1
- Ninja 1.13.1

## Results

The nCine version is the `master` branch as of today, compiled with `CMAKE_BUILD_TYPE=Release`, but left all the rest untouched, which means it was compiled with the GLFW backend, with `NCINE_BUILD_TESTS` set to `ON`, and `NCINE_BUILD_UNIT_TESTS` and `NCINE_BUILD_ANDROID` both set to `OFF`.
The tests have been conducted by running the compilation process multiple times and recording the best timings.

This time I've only timed the build phase, using `ninja` on all devices. For the laptops, I’ve repeated the test on both the power saving and the performance profile (the former limits CPU frequency and power draw to extend battery life, while the latter allows higher sustained clocks and power consumption).

### Tables and charts

#### Arch Linux

|              |  Asus   |  Alurin  |  Mini   |
| ------------ | -------:| --------:| -------:|
|  Power Save  | 22.359s | 47.213s  |         |
| Performance  | 14.379s | 17.725s  | 37.33s  |

![Compilation Chart 2025](/images/compilation_chart_2025.png "Compilation Chart 2025")

### Conclusions

It's quite strange to see the Ryzen 6800HS and the Intel i5-1235U so close in Performance mode, maybe compiling C++ code is not the most parallelizable process in the world. :sweat_smile:
With other benchmarks, like Cinebench, this Ryzen CPU dominates the smaller i5 on the multi-core test, with around double the performance.

It seems that power profiles are tuned quite differently between the two laptops, seeing how much the Intel one loses in Power Save.

I was a bit disappointed by the M1, on paper it should be a bit faster than the Intel on the CPU side with Cinebench, but it loses in my tests. Maybe it's just that I'm comparing apples and oranges, with a different OS, environment, and compiler.

I hope you have found this second benchmark post at least as interesting as the first one. :wink:
