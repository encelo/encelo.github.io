---
layout: post
title: nCine Compilation Benchmark
subtitle: How long does it take to compile the nCine?
tags: [nCine]
---

Are you curious about the time I spend to compile the nCine? :smile:  
In this post I'm going to show how much time is needed to compile the engine on different platforms.

Let's start with the hardware and the software I've used for my tests.

## Hardware and Software

### Xiaomi Mi Notebook Pro

**Hardware**
- Intel Core i7-8550U
- 16GB RAM, DDR4 2400 MHz, dual channel
- Samsung PM961 NVMe SSD, 256GB
- NVIDIA GeForce MX150, 2GB GDDR5

**OS**
- Microsoft Windows 10 Professional (x64), Build 16299.248
- Arch Linux x86_64, Linux 4.15.3

### Xiaomi Mi Notebook Air 13 2016

**Hardware**
- Intel Core i5-6200U
- 8GB RAM, DDR4 2133 MHz, single channel
- Samsung PM951 NVMe SSD, 256GB
- NVIDIA GeForce 940MX, 2GB GDDR5

**OS**
- Microsoft Windows 10 Professional (x64), Build 16299.248
- Arch Linux x86_64, Linux 4.15.3

### Chuwi Hi 10

**Hardware**
- Intel Atom x5-Z8300
- 4G RAM, DDR3 1600 MHz, single channel
- Hynix HCG8e eMMC 5.1, 64GB

**OS**
- Microsoft Windows 10 Home (x64), Build 16299.248

### Compilers and Tools

**Windows**
- VS2017 15.5.6 (MSVC 19.12.25835)
- CMake 3.10.2 64bit

**MSYS2 and Arch Linux**
- GCC 7.3.0
- CMake 3.10.2
- Ninja 1.8.2

## Results

The nCine version is `2018.02.r183-2d21790`, compiled with GLFW, `NCINE_BUILD_TESTS` set to `ON`, `NCINE_BUILD_UNIT_TESTS` and `NCINE_BUILD_ANDROID` both set to `OFF`.
The tests have been conducted by running the compilation process multiple times and recording the best timings.

In the _Configure_ phase CMake is invoked in order to configure the project and generate either a Visual Studio solutions, or Makefiles, or Ninja files.
In the _Build_ phase CMake is invoked with the `--build` option in order to build the project. It takes advantage of multiple cores to compile only if using Ninja or MSVC with the `/MP` option. When using standard makefiles I have additionally provided the timings of invoking `make -jN` for the multicore compilation.

### Tables and Charts

#### MSVC

|                      | Mi Pro  | Mi Air | Chuwi   |
| -------------------- | -------:| ------:| -------:|
| Configure            | 14.20s  | 12.02s |         |
| Build                | 22.23s  | 32.93s |         |
| Build with `/MP`     | 14.87s  | 26.48s |         |

![MSVC chart](/images/MSVC_chart.png "MSVC chart")


#### MSYS2

|                      | Mi Pro  | Mi Air | Chuwi   |
| -------------------- | -------:| ------:| -------:|
| Configure Ninja      | 8.13s   | 12.33s | 40.90s  |
| Build Ninja          | 7.13s   | 18.84s | 55.71s  |
| Configure Make       | 14.14s  | 19.75s | 61.13s  |
| Build Make           | 46.80s  | 66.12s | 226.53s |
| Build Make `-j`      | 12.65s  | 30.59s | 84.90s  |

![MSYS2 chart](/images/MSYS2_chart.png "MSYS2 chart")

#### Arch Linux

|                      | Mi Pro  | Mi Air | Chuwi   |
| -------------------- | -------:| ------:| -------:|
| Configure Ninja      | 1.03s   | 1.31s  |         |
| Build Ninja          | 3.68s   | 7.85s  |         |
| Configure Make       | 1.25s   | 1.58s  |         |
| Build Make           | 13.56s  | 17.31s |         |
| Build Make `-j`      | 4.35s   | 9.17s  |         |

![Arch Linux chart](/images/ArchLinux_chart.png "Arch Linux chart")

### Conclusions

The Mi Notebook Pro, with its Kaby Lake R featuring 4C/8T, achieves more than double the performance of the smaller Mi Notebook Air 13 when compiling with Ninja or `make -j8` on all platforms. On Windows the speed-up is slightly less, but still very noticeable.  
The Mi Notebook Pro comes with a faster SSD and dual channel RAM which surely have affected timings. The little and fanless Chuwi Hi 10 comes last when talking about performances, but it can still configure and build the project in under 100 seconds with Ninja. :smile:

Speaking about the configure phase, it strangely takes a bit more time on the Mi Notebook Pro compared to the slower Mi Notebook Air on Windows.
It is also interesting to note that the Ninja generator in CMake is faster than the Makefiles one on all platforms, even if the difference on Linux is barely noticeable.  
Speaking about Linux, I wasn't prepared for such a big margin when compared with Windows. Can it be a matter of Ext4 vs NTFS? Or maybe a slowdown when invoking commands through the MSYS2 console? I'm not sure of the reasons but lucky me for I usually develop on Linux and then test on the supported platforms. :sweat_smile:

I hope you have found this post interesting even if it wasn't about a development update. They will come back soon so stay tuned! :wink:
