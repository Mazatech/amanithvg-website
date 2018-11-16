---
layout: default
title: "Objects parameters"
date: 2017-01-01 08:00:00 +0100
chapter: 6
categories: [vgapi]
headline: "Openvg API, set and get objects parameters"
image: "amanithvg-logo.png"
keywords: "openvg api objects parameters set get"
---

# Object Parameters

## Object Parameters Set/Get API [5.3]

Objects that are referenced using a `VGHandle` (`VGImage`, `VGPaint`, `VGPath`, `VGFont`, and `VGMaskLayer` objects) may have their parameters set and queried using a number of `vgSetParameter` and `vgGetParameter` functions. The semantics of these functions (including the handling of invalid count values) are similar to those of the
`vgGet` and `vgSet` functions.

```c
void vgSetParameterf(VGHandle obj, VGint paramType, VGfloat val)
```
```c
void vgSetParameteri(VGHandle obj, VGint paramType, VGfloat val)
```
```c
void vgSetParameterfv(VGHandle obj, VGint paramType, VGint count, const VGfloat* val)
```
```c
void vgSetParameteriv(VGHandle obj, VGint paramType, VGint count, const VGint* val)
```
```c
VGfloat vgGetParameterf(VGHandle obj, VGint paramType)
```
```c
VGint vgGetParameteri(VGHandle obj, VGint paramType)
```
```c
VGint vgGetParameterVectorSize(VGHandle obj, VGint paramType)
```
```c
void vgGetParameterfv(VGHandle obj, VGint paramType, VGint count, VGfloat* val)
```
```c
void vgGetParameteriv(VGHandle obj, VGint type, VGint count, VGint* val)
```

---
