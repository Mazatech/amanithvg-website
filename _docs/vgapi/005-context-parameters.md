---
layout: default
title: "Context parameters"
date: 2018-11-01 08:00:00 +0100
chapter: 5
categories: [vgapi]
headline: "Openvg API, set and get context parameters"
image: "amanithvg-logo.png"
keywords: "openvg api context parameters set get"
---

# Context Parameters

## Context Parameter Set/Get API [5.2]

Each `vgGet` or `vgSet` function has four variants, depending on the data type of the value being set, differentiated by a suffix: `i` for scalar integral values, `f` for scalar floating-point values, and `iv` and `fv` for vectors of integers and floating-point values, respectively.  
The vector variants may also be used to set scalar values using a count of 1. When setting a value of integral type using a floating-point `vgSet` variant (ending with `f` or `fv`), or retrieving a floating-point value using an integer `vgGet` function (ending with `i` or `iv`), the value is converted to an integer using a mathematical floor operation.

```c
void vgSetf(VGParamType paramType, VGfloat val)
```
```c
void vgSeti(VGParamType paramType, VGint val)
```
```c
void vgSetfv(VGParamType paramType, VGint count, const VGfloat* val)
```
```c
void vgSetiv(VGParamType paramType, VGint count, const VGint* val)
```
```c
VGfloat vgGetf(VGParamType paramType)
```
```c
VGint vgGeti(VGParamType paramType)
```
```c
VGint vgGetVectorSize(VGParamType paramType)
```
```c
void vgGetfv(VGParamType paramType, VGint count, VGfloat* val)
```
```c
void vgGetiv(VGParamType paramType, VGint count, VGint* val)
```

---

### Context Parameters [5.2.1]

The possible values of paramType from enumeration `VGParamType` are shown below, with the legal values for val. Default value is indicated in <span class="ovg_default">YELLOW</span>.

| Parameter name | Parameter type | Possible values / Notes |
| -------------- | -------------- | ----------------------- |
| `VG_MATRIX_MODE` | `VGMatrixMode` | <span class="ovg_default">VG_MATRIX_PATH_USER_TO_SURFACE</span> <br> `VG_MATRIX_IMAGE_USER_TO_SURFACE`<br> `VG_MATRIX_FILL_PAINT_TO_USER`<br> `VG_MATRIX_STROKE_PAINT_TO_USER`<br> `VG_MATRIX_GLYPH_USER_TO_SURFACE` |
| `VG_FILL_RULE` | `VGFillRule` | <span class="ovg_default">VG_EVEN_ODD</span> <br> `VG_NON_ZERO` |
| `VG_IMAGE_QUALITY` | `VGImageQuality` | `VG_IMAGE_QUALITY_NONANTIALIASED`<br> <span class="ovg_default">VG_IMAGE_QUALITY_FASTER</span> <br> `VG_IMAGE_QUALITY_BETTER` |
| `VG_RENDERING_QUALITY` | `VGRenderingQuality` | `VG_RENDERING_QUALITY_NONANTIALIASED`<br> `VG_RENDERING_QUALITY_FASTER`<br> <span class="ovg_default">`VG_RENDERING_QUALITY_BETTER`</span> |
| `VG_BLEND_MODE` | `VGBlendMode` | `VG_BLEND_SRC`<br> <span class="ovg_default">VG_BLEND_SRC_OVER</span><br> `VG_BLEND_DST_OVER`<br> `VG_BLEND_SRC_IN`<br> `VG_BLEND_DST_IN`<br> `VG_BLEND_MULTIPLY`<br> `VG_BLEND_SCREEN`<br> `VG_BLEND_DARKEN`<br> `VG_BLEND_LIGHTEN`<br> `VG_BLEND_ADDITIVE` |
| `VG_IMAGE_MODE` | `VGImageMode` | <span class="ovg_default">VG_DRAW_IMAGE_NORMAL</span> <br> `VG_DRAW_IMAGE_MULTIPLY`<br> `VG_DRAW_IMAGE_STENCIL` |
| `VG_SCISSOR_RECTS` | `VGint*` | Format is:<br> `{ xMin, yMin, width, height, ... }`<br> Rects with width <= 0 or height <= 0 are ignored<br> <span class="ovg_default">{ }</span> |
| `VG_COLOR_TRANSFORM` | `VGboolean` | `VG_TRUE`<br> <span class="ovg_default">VG_FALSE</span> |
| `VG_COLOR_TRANSFORM_VALUES` | `VGfloat[8]` | Format is:<br> `{ Rscl, Gscl, Bscl, Ascl,`<br> `Rbias, Gbias, Bbias, Abias }`<br> Scale parameters are clamped to:<br> `[-127.0, +127.0]`<br> Bias parameters are clamped to:<br> `[-1.0, +1.0]`<br> <span class="ovg_default">{ 1.0f, 1.0f, 1.0f, 1.0f, 0.0f, 0.0f, 0.0f, 0.0f }</span> |
| `VG_STROKE_LINE_WIDTH` | `VGfloat` | Any positive value is valid (a value <=0 prevents stroking from taking place)<br> <span class="ovg_default">1.0f</span> |
| `VG_STROKE_CAP_STYLE` | `VGCapStyle` | <span class="ovg_default">VG_CAP_BUTT</span> <br> `VG_CAP_ROUND`<br> `VG_CAP_SQUARE` |
| `VG_STROKE_JOIN_STYLE` | `VGJoinStyle` | <span class="ovg_default">VG_JOIN_MITER</span> <br> `VG_JOIN_ROUND`<br> `VG_JOIN_BEVEL` |
| `VG_STROKE_MITER_LIMIT` | `VGfloat` | Miter limit values <=1 are silently clamped to 1<br> <span class="ovg_default">4.0f</span> |
| `VG_STROKE_DASH_PATTERN` | `VGfloat*` | Format is:<br> `{ on1, off1, on2, off2, ... }`<br> If the dash pattern has length 0, dashing is not performed. If the dash pattern has an odd number of elements, the final element is ignored.<br> <span class="ovg_default">{ }</span> |
| `VG_STROKE_DASH_PHASE` | `VGfloat` | Any float value is valid<br> <span class="ovg_default">0.0f</span> |
| `VG_STROKE_DASH_PHASE_RESET` | `VGboolean` | `VG_TRUE`<br> <span class="ovg_default">VG_FALSE</span> |
| `VG_TILE_FILL_COLOR` | `VGfloat[4]` | Format is:<br> `{ red, green, blue, alpha } sRGBA`<br> Values outside the [0, 1] range are interpreted as the nearest endpoint of the range<br> <span class="ovg_default">{ 0.0f, 0.0f, 0.0f, 0.0f }</span> |
| `VG_CLEAR_COLOR` | `VGfloat[4]` | Format is:<br> `{ red, green, blue, alpha } sRGBA`<br> Values outside the [0, 1] range are interpreted as the nearest endpoint of the range<br> <span class="ovg_default">{ 0.0f, 0.0f, 0.0f, 0.0f }</span> |
| `VG_GLYPH_ORIGIN` | `VGfloat[2]` | Format is:<br> `{ xOrigin, yOrigin }`<br> <span class="ovg_default">{ 0.0f, 0.0f }</span> |
| `VG_MASKING` | `VGboolean` | `VG_TRUE`<br> <span class="ovg_default">VG_FALSE</span> |
| `VG_SCISSORING` | `VGboolean` | `VG_TRUE`<br> <span class="ovg_default">VG_FALSE</span> |
| `VG_PIXEL_LAYOUT ` | `VGPixelLayout` | <span class="ovg_default">VG_PIXEL_LAYOUT_UNKNOWN</span> <br> `VG_PIXEL_LAYOUT_RGB_VERTICAL`<br> `VG_PIXEL_LAYOUT_BGR_VERTICAL`<br> `VG_PIXEL_LAYOUT_RGB_HORIZONTAL`<br> `VG_PIXEL_LAYOUT_BGR_HORIZONTAL` |
| `VG_FILTER_FORMAT_LINEAR` | `VGboolean` | `VG_TRUE`<br> <span class="ovg_default">VG_FALSE</span> |
| `VG_FILTER_FORMAT_PREMULTIPLIED` | `VGboolean` | `VG_TRUE`<br> <span class="ovg_default">VG_FALSE</span> |
| `VG_FILTER_CHANNEL_MASK` | `VGbitfield` | Any combination of:<br> `VG_RED, VG_GREEN, VG_BLUE, VG_ALPHA` <br> <span class="ovg_default">VG_RED \| VG_GREEN \| VG_BLUE \| VG_ALPHA</span> |
{:.rwd-table .rwd-table-contextParameters}

---

### Read-Only Context Parameters [5.2.1]

Some context parameters are read-only, so they cannot be set through `vgSet`(`i`/`f`/`iv`/`fv`) functions.

| Parameter name | Parameter type | AmanithVG | Notes |
| -------------- | -------------- | --------- | ----- |
| `VG_MAX_SCISSOR_RECTS` | `VGint` | `VG_MAXINT` | All implementations must support at least 32 scissor rectangles |
| `VG_MAX_DASH_COUNT` | `VGint` | `VG_MAXINT` | All implementations must must support at least 16 dash segments (8 on/off pairs) |
| `VG_MAX_KERNEL_SIZE` | `VGint` | `255` | All implementations must define this value to be an integer no smaller than 7 |
| `VG_MAX_SEPARABLE_KERNEL_SIZE` | `VGint` | `255` | All implementations must define this value to be an integer no smaller than 15 |
| `VG_MAX_GAUSSIAN_STD_DEVIATION` | `VGfloat` | `128` | All implementations must define this value to be no smaller than 16 |
| `VG_MAX_COLOR_RAMP_STOPS` | `VGint` | `VG_MAXINT` | All implementations must support at least 32 stops |
| `VG_MAX_IMAGE_WIDTH` | `VGint` | `4196` | Configurable on AmanithVG.<br> All implementations must define this value to be an integer no smaller than 256 |
| `VG_MAX_IMAGE_HEIGHT` | `VGint` | `4196` | Configurable on AmanithVG.<br> All implementations must define this value to be an integer no smaller than 256 |
| `VG_MAX_IMAGE_PIXELS` | `VGint` | `17606416` | All implementations must define this value to be an integer no smaller than 65536 |
| `VG_MAX_IMAGE_BYTES` | `VGint` | `70425664` | All implementations must define this value to be an integer no smaller than 65536 |
| `VG_MAX_FLOAT` | `VGfloat` | `3.402823466e+38` | All implementations must define this value to be at least 10^10 |
{:.rwd-table .rwd-table-roContextParameters}

---
