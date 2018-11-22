---
layout: default
title: "Errors"
date: 2017-01-01 08:00:00 +0100
chapter: 2
categories: [vgapi]
headline: "Openvg API, error codes"
image: "amanithvg-logo.png"
keywords: "openvg api error codes"
---

# Errors

## Errors [4.1]

The `vgGetError` function returns the oldest error code provided by an OpenVG function call, error codes and their numerical values are defined by the `VGErrorCode` enumeration.  
After a call to `vgGetError`, the error code is cleared to `VG_NO_ERROR`.

| Error code | Notes |
| ---------- | ----- |
| `VG_NO_ERROR` | Defined as 0 |
| `VG_BAD_HANDLE_ERROR` | - |
| `VG_ILLEGAL_ARGUMENT_ERROR` | - |
| `VG_OUT_OF_MEMORY_ERROR` | ALL OpenVG functions may signal `VG_OUT_OF_MEMORY_ERROR` |
| `VG_PATH_CAPABILITY_ERROR` | - |
| `VG_UNSUPPORTED_IMAGE_FORMAT_ERROR` | - |
| `VG_UNSUPPORTED_PATH_FORMAT_ERROR` | - |
| `VG_IMAGE_IN_USE_ERROR` | Passing an image that is currently the rendering target to any OpenVG function (excluding `vgGetParameter` and `vgDestroyImage`) will result in a such error |
| `VG_NO_CONTEXT_ERROR` | If no context is current at the time `vgGetError` is called, this error code is returned |
{:.rwd-table .rwd-table-errors}

---
