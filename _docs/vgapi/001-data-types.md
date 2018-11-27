---
layout: default
title: "Data Types"
date: 2018-11-01 08:00:00 +0100
chapter: 1
categories: [vgapi]
headline: "Openvg API, data types"
image: "amanithvg-logo.png"
keywords: "openvg api data types primitive handle"
---

# Data Types

## Primitive data types [3.2]

 | Data type name | Size | Values range |
 | -------------- | ---- | -------------|
 | `VGbyte` | `1 byte` | `[ -128, 127 ]` |
 | `VGubyte` | `1 byte` | `[ 0, 255 ]` |
 | `VGshort` | `2 byte` | `[ -32768, 32767 ]` |
 | `VGint` | `4 byte` | `[ -(2^31), 2^31 - 1 ]` |
 | `VGuint` | `4 byte` | `[ 0, 2^32 - 1 ]` |
 | `VGfloat` | `4 byte` | `IEEE 754 Standard` | 
 | `VGboolean` | `4 byte` | `[ VG_FALSE(0), VG_TRUE(1) ]` |
 | `VGbitfiled` | `4 byte` | `[ 0, 2^32 - 1 ]` |
{:.rwd-table .rwd-table-primitiveDataTypes}

---

## Handle-based data types [3.6]

Images, paint objects, and paths are accessed using opaque handles.  
Handles employ reference count semantics: if a handle is in use, a request to destroy it prevents the handle from being used further by the application, but allows it to continue to be used internally by the OpenVG implementation until it is no longer referenced.  
`VG_INVALID_HANDLE` (defined as `(VGHandle)0`) represents an invalid `VGHandle` that is used as an error return value from functions that return a `VGHandle`.


| Data type name  | Description |
| --------------- | ----------- |
| `VGHandle` | `VGuint` |
| `VGPath` | a `VGHandle` referencing path data |
| `VGImage` | a `VGHandle` referencing image data |
| `VGMaskLayer` | a `VGHandle` referencing mask data |
| `VGFont` | a `VGHandle` referencing path data |
| `VGPaint` | a `VGHandle` referencing paint specification |
{:.rwd-table .rwd-table-handleBasedDataTypes}

---
