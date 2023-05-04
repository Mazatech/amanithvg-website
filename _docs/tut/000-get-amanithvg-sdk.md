---
layout: default
title: "Get AmanithVG SDK"
date: 2018-11-01 08:00:00 +0100
chapter: 0
categories: [tut]
headline: "Getting started with AmanithVG SDK: download, installation and setup toolchain"
image: "amanithvg-logo.png"
keywords: "amanithvg tutorial 00 get start sdk setup openvg"
---

# Getting started

## Install CMake

Install CMake 3.12+ tool following instructions at: [https://cmake.org/install](https://cmake.org/install/).

AmanithVG SDK uses CMake to generate makefiles and Xcode/VStudio solutions.

___

## Download AmanithVG SDK from Github

Create an empty directory, enter it, then use Git or checkout with SVN using the web URL:

```
git clone https://github.com/Mazatech/amanithvg-sdk.git
```

---

## Tutorials

Within the `amanithvg-sdk` directory, you'll find `/tutorials` folder containing tutorials on how to use AmanithVG (and the OpenVG API) on several platforms/architectures.

Use `CMake` with the following options:

### OpenVG Engine

Choose a single backend using:

```
-DENGINE_SRE=1 // Software rendering engine, or
-DENGINE_GLE=1 // OpenGL (ES) rendering engine
```

### Toolchain & Platform

Choose a toolchain and platform using:

```
// Windows (Visual Studio)
// -----------------------
-DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/win_x86.cmake     // Windows x86, or
-DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/win_x86_64.cmake  // Windows x86_64

// Generators
-G "Visual Studio 17 2022" -A [platform]   // Visual Studio 2022 solution, or
-G "Visual Studio 16 2019" -A [platform]   // Visual Studio 2019 solution, or
-G "Visual Studio 15 2017 [arch]"          // Visual Studio 2017 solution, or
-G "NMake Makefiles"                       // NMake
```

```
// MacOS X (Xcode)
// ---------------
-DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/osx_ub.cmake  // Universal Binary (arm64, x86_64)

// Generators
-G "Xcode"           // Xcode project
```

```
// Linux X11 (gcc)
// ---------------
-DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/linux_x86.cmake     // Linux x86, or
-DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/linux_x86_64.cmake  // Linux x86_64

// Generator
-G "Unix Makefiles"  // makefiles
```

```
// iOS (Xcode)
// -----------
-DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/ios_ub.cmake  // Universal Binary (arm64, x86_64)

// Generator
-G "Xcode"  // Xcode project
```

---

## Some CMake examples

### Windows
```
// AmanithVG GLE backend, Windows x86_64, Visual Studio 2022 solution
<open x64 Native Tools Command Prompt for VS 2022>
cmake -DENGINE_GLE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/win_x86_64.cmake --no-warn-unused-cli -G "Visual Studio 17 2022" -A x64 .
<open the generated .sln solution>

// AmanithVG SRE backend, Windows x86_64, Visual Studio 2019 solution
<open x64 Native Tools Command Prompt for VS 2019>
cmake -DENGINE_SRE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/win_x86_64.cmake --no-warn-unused-cli -G "Visual Studio 16 2019" -A x64 .
<open the generated .sln solution>

// AmanithVG GLE backend, Windows x86, Visual Studio 2017 nmake
<open x86 Native Tools Command Prompt for VS 2017>
cmake -DENGINE_GLE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/win_x86.cmake --no-warn-unused-cli -G "NMake Makefiles" .
nmake
```

### MacOS X
```
// AmanithVG SRE backend, MacOS X, Xcode project
<open a command prompt>
cmake -DENGINE_SRE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/osx_ub.cmake --no-warn-unused-cli -G "Xcode" .
make

// AmanithVG GLE backend, MacOS X, Xcode project
<open a command prompt>
cmake -DENGINE_GLE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/osx_ub.cmake --no-warn-unused-cli -G "Xcode" .
<open the generated .xcodeproj project>
```

### Linux
```
// AmanithVG SRE backend, Linux x86_64, standard Makefile
<open a command prompt>
cmake -DENGINE_SRE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/linux_x86_64.cmake --no-warn-unused-cli -G "Unix Makefiles" .
make

// AmanithVG GLE backend, Linux x86_64, standard Makefile
<open a command prompt>
cmake -DENGINE_GLE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/linux_x86_64.cmake --no-warn-unused-cli -G "Unix Makefiles" .
make
```

### iOS
```
// AmanithVG SRE backend, iOS, Xcode project
<open a command prompt>
cmake -DENGINE_SRE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/ios_ub.cmake --no-warn-unused-cli -G "Xcode" .
<open the generated .xcodeproj project>

// AmanithVG GLE backend, iOS, Xcode project
<open a command prompt>
cmake -DENGINE_GLE=1 -DCMAKE_TOOLCHAIN_FILE=./CMake/toolchain/ios_ub.cmake --no-warn-unused-cli -G "Xcode" .
<open the generated .xcodeproj project>
```

---

## Android tutorials

Tutorials for Android can be compiled directly with [Android Studio](https://developer.android.com/studio) (you don't need to use CMake).

Open project located in `<tutorial_dir>/platform/android`

AmanithVG SRE (software rendering) is the default, you can switch to AmanithVG GLE (OpenGL rendering), editing `<tutorial_dir>/platform/android/app/src/main/res/values/bools.xml`:

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <bool name="sreBackend">false</bool>
</resources>
```

---

## iOS tutorials

iOS tutorials can be compiled using Xcode only.
Once you have generated the Xcode project (through CMake), plug your iOS device in and open the .xcodeproj file.

| &nbsp; |
| :---: |
| *Open the .xcodeproj file* |
{:.tbl_images .tut00_setup001}

Then select the iOS device and check project properties.
Select a valid developer certificate for the signing process.

| &nbsp; |
| :---: |
| *Select a valid developer certificate, before* |
{:.tbl_images .tut00_setup002}

| &nbsp; |
| :---: |
| *Select a valid developer certificate, after* |
{:.tbl_images .tut00_setup003}

Select the actual tutorial target, in order to run it.

| &nbsp; |
| :---: |
| *Select a tutorial target* |
{:.tbl_images .tut00_setup004}

---
