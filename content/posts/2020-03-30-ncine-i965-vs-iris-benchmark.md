---
date: "2020-03-30"
title: nCine Intel Mesa 20 Driver Benchmark
description: i965 vs iris
tags: [ nCine, Benchmarks, Rendering ]
---

Today I upgraded my Arch Linux workstation with pacman as I usually do every day and a little surprise was waiting for me.
After a long time in `[testing]`, [Mesa 20](https://www.archlinux.org/packages/extra/x86_64/mesa/) came out of the `[extra]` repository, ready to be installed.

This [release](https://lists.freedesktop.org/archives/mesa-dev/2020-February/224132.html) brings a ton of fixes and new features, alongside the new [iris](https://xdc2018.x.org/slides/optimizing-i965-for-the-future.pdf) driver for modern Intel integrated GPUs based on Gallium3D.

I have conducted some benchmarks on my Intel Core i7-8550U with Intel HD Graphics 620 (GT2), an Intel Gen 9.5 GPU.

### Setup

I have compiled the nCine in release and disabled V-Sync, then I ran a script to minimize CPU throttling and frequency variations:

```bash
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo 800000 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq
echo 800000 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq
[repeat for remaining cores, cpu1 to cpu7]
```

The `performance` [governor](https://wiki.archlinux.org/index.php/CPU_frequency_scaling#Scaling_governors) should keep the CPU running at the same frequency all the time while the minimum frequency of 800 Mhz is set as both the minimum and maximum overall frequency. Those steps should both prevent the CPU from overheating and possibly highlight the reduced CPU load of `iris`.

To choose between the two drivers I used the `MESA_LOADER_DRIVER_OVERRIDE` environment variable. I set it to `i965` for the legacy one or unset the variable for the new one, as it is now the default.

To confirm which driver was in use during a test I set the `LIBGL_DEBUG` environment variable to `verbose`.

### Tests

Let's see how many frames can be rendered over a time of five seconds.

| Test                               |  i965  |  iris  |  Ratio  |
| ---------------------------------- | ------:| ------:| -------:|
| `apptest_camera` (800 Mhz)         |  1665  |  1659  |  0.99x  |
| `apptest_camera`                   |  4532  |  4646  |  1.02x  |
| `apptest_meshsprites` (800 Mhz)    |  1281  |  1294  |  1.01x  |
| `apptest_meshsprites`              |  3451  |  3515  |  1.01x  |
| `apptest_particles_100x` (800 Mhz) |  385   |  387   |  1.00x  |
| `apptest_particles_100x`           |  509   |  508   |  1.00x  |

### Conclusions

It seems that for the nCine kind of workload, 2D sprites and low vertex count, there is practically no difference between the two drivers on my machine. The results do not change when the CPU frequency is capped at its minimum.

This is not necessarily a bad thing. I would have been glad to see some performance uplift with those 2D tests but I'm glad to see that `iris` does not seem to suffer at all from being a lot less mature than `i965`.
