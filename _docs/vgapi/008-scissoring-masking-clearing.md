---
layout: default
title: "Scissoring, masking, clearing"
date: 2018-11-01 08:00:00 +0100
chapter: 8
categories: [vgapi]
headline: "Openvg API, Scissoring, masking and clearing"
image: "amanithvg-logo.png"
keywords: "openvg api scissoring mask masking clear"
---

# Scissoring, Masking, and Clearing

All drawing is clipped (restricted) to the bounds of the drawing surface, and may be further clipped to the interior of a set of scissoring rectangles. If available, a mask is applied for further clipping and to create soft edge and partial transparency effects.

---

## Scissoring [7.1]

Drawing may be restricted to the union of a set of scissoring rectangles. Each scissoring rectangle is specified as an integer 4-tuple of the form (minX, minY, width, height). The scissoring region is defined as the union of all the specified rectangles. The rectangles as specified need not be disjoint.
Scissoring is enabled when the parameter `VG_SCISSORING` has the value `VG_TRUE`. Scissoring may be disabled by calling `vgSeti` with a `paramType` argument of `VG_SCISSORING` and a value of `VG_FALSE`.

```c
#define NUM_RECTS 2
/* { Min X, Min Y, Width, Height } 4-Tuples */
VGint coords[4 * NUM_RECTS] = {
    20, 30, 100, 200,
    50, 70, 80, 80
};
vgSetiv(VG_SCISSOR_RECTS, 4 * NUM_RECTS, coords)
/* enable scissoring */
vgSeti(VG_SCISSORING, VG_TRUE)
```

---

## Masking [7.2]

All drawing operations may be modified by a drawing surface mask (also known as an alpha mask), which is a separate implementation-internal buffer defining an additional coverage value at each sample of the drawing surface. Masking is enabled when a mask is present for the drawing surface and the `VG_MASKING` parameter has the value `VG_TRUE`.
Masking may be disabled by calling `vgSeti` with a parameter of `VG_MASKING` and a value of `VG_FALSE`. Alpha mask may be manipulated by the `vgMask` function: the function modifies the mask values according to a given operation, possibly using coverage values taken from a mask layer or bitmap image. In addition, may be modified by the The `vgRenderToMask` function: the function modifies the current mask by applying the given operation to the set of coverage values associated with the rendering of the given path.

```c
void vgMask(VGHandle mask,
            VGMaskOperation op, 
            VGint x, VGint y,
            VGint width, VGint height)
```
```c
void vgRenderToMask(VGPath path, VGbitfield paintMode, VGMaskOperation op)
```
```c
VGMaskLayer vgCreateMaskLayer(VGint width, VGint height)
```
```c
void vgDestroyMaskLayer(VGMaskLayer masklayer)
```
```c
void vgFillMaskLayer(VGMaskLayer masklayer,
                     VGint x, VGint y,
                     VGint width, VGint height,
                     VGfloat val)
```
```c
void vgCopyMask(VGMaskLayer masklayer,
                VGint x, VGint y, 
                VGint sx, VGint sy, 
                VGint width, VGint height, 
                VGfloat val)
```

The `VGMaskOperation` enumeration defines the set of possible operations that may be used to modify a mask.

`Amask` = input mask, `Aprev` = previous mask

| Operation | Resulting mask |
| --------- | -------------- |
| `VG_CLEAR_MASK` | `0` |
| `VG_FILL_MASK` | `1` |
| `VG_SET_MASK` | `Amask` |
| `VG_UNION_MASK` | `1 - (1 - Amask) * (1 - Aprev)` |
| `VG_INTERSECT_MASK` | `Amask * Aprev` |
| `VG_SUBTRACT_MASK` | `Aprev * (1 - Amask)` |
{:.rwd-table .rwd-table-masking}

---

## Fast Clearing [7.3]

The `vgClear` function fills the portion of the drawing surface intersecting the rectangle extending from pixel (x, y) and having the given width and height with a constant color value, taken from the `VG_CLEAR_COLOR` parameter.

```c
void vgClear(VGint x, VGint y, VGint width, VGint height)
```

---
