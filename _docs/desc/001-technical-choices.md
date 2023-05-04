---
layout: default
title: "Technical choices"
date: 2018-11-01 08:00:00 +0100
chapter: 1
categories: [desc]
headline: "AmanithVG technical choices and supported platforms"
image: "amanithvg-logo.png"
keywords: "amanithvg sre gle engines 2d vector graphics techincal engines supported platforms openvg"
---

# Technical choices

AmanithVG has been designed to run everywhere. To accomplish this task the development process has followed strict technical choices: 

 * Use of pure ANSI C language, avoiding the use of any third party libraries.
 
 * Use of advanced caching techniques at different levels (curve flattening, stroking).
 
 * High and easy customization capability through simple hardware independent thresholds.
 
 * Modular and flexible architecture to simplify new extensions integration.
 
 * Fast and robust fixed-point polygon triangulation (GLE).
 
 * Minimization of the CPU usage, moving the most complex tasks on the 3D chipset / GPU (GLE).
 
 * Dynamic rendering pipelines, according to underlying 3D hardware features / extensions (GLE).
 
 * Use of few smart software fallbacks on very old hardware (GLE).
 
 * Fast and robust fixed-point polygon rasterizer (SRE).
 
 * Use of dedicated optimized scanline fillers, instead of a slow monolithic pixel pipeline (SRE).

---

## Supported platforms

AmanithVG can be (cross)compiled for the following target platforms:

 * Windows XP / Vista / 7 / 8 / 10 / 11, on x86, x86_64, compilable with:
   * Microsoft (R) C/C++ Optimizing Compiler Version 19.16.27043 (or higher) for x86
   * Microsoft (R) C/C++ Optimizing Compiler Version 19.16.27043 (or higher) for x86_64
   * mingw-w64 (gcc 6.3.0 or higher) for x86 and x86_64

 * MacOS X 10.15 or higher, (Universal Binary), on ARM v8a, x86_64 compilable with:
   * Xcode version 14.0 or higher (clang-1400 or higher)

 * Linux 2.6.x or higher, on x86, x86_64, ARM v5, ARM v6 (with or without VFPv2), ARM v7a, ARM v8a, mipsel, mips64el, ppc64el, compilable with:
   * gcc version 6.3.0 or higher
   
 * Windows CE / Mobile, on ARM v6 (with or without VFPv2), v7 (VFPv3) compilable with:
   * arm-mingw32ce version 2020-03-23-amd64 (gcc 9.3.0)

 * iOS 11 or higher (Universal Binary, bitcode support), on ARM v8a, x86_64 (simulator) compilable with:
   * Xcode version 14.0 or higher (clang-1400 or higher)

 * Android, on ARM v7a, ARM v8a, x86, x86_64, compilable with:
   * Android NDK 25c or higher (Android clang version 14.0.7 or higher)

 * QNX 6.6 on x86, ARM v7a, compilable with:
   * QNX Momentics Tool Suite 6.6 (gcc 4.7.3)

---
