---
layout: default
title: "Tut. 10 - Masking"
date: 2017-01-01 08:00:00 +0100
chapter: 10
categories: [tut]
---

# Tutorial 10: Masking

All drawing operations (e.g. `vgDrawPath` and `vgDrawImage`) may be modified by a drawing surface mask (also known as an *alpha mask* for historical reasons), which is a separate implementation-internal buffer defining an additional coverage value at each sample of the drawing surface. The values from this buffer modify the coverage value computed by the rasterization stage of the OpenVG pipeline.

Let's have a look at how we can manipulate alpha mask.

---

## OpenVG alpha mask

The drawing surface mask may be thought of as a single-channel image with the same size as the current drawing surface; if the drawing surface size changes, the drawing surface mask associated with it is resized accordingly.
As a convention, we set coverage values to stay within a `[0; 1]` range; the actual bit depth used for computation is implementation-dependent (AmanithVG uses `8bit`, so a maximum of `256` alpha levels). Initially, the mask has the value of 1 at every pixel.

The mask coverage values are multiplied by the corresponding coverage values of each primitive being drawn in the clipping and masking stage of the OpenVG rendering pipeline.
Masking is enabled when a mask is present for the drawing surface (see the `vgPrivSurfaceCreateMZT` function, [EGL like interface]({{site.url}}/docs/desc/003-egl.html) section) and the `VG_MASKING` parameter has the value `VG_TRUE`. Masking may be disabled by calling `vgSeti` with a parameter of `VG_MASKING` and a value of `VG_FALSE`.

Surface mask can be manipulated, regardless of the value of `VG_MASKING`, by two main functions: `vgMask` and `vgRenderToMask`.

---

##  vgMask

The vgMask function modifies the drawing surface mask values according to a given `operation`, using coverage values taken from a bitmap image given by the `mask` parameter.
The affected region is the intersection of the drawing surface bounds with the rectangle extending from pixel `(x, y)` of the drawing surface and having the given `width` and `height` in pixels.
Mask pixels starting at `(0, 0)` are used, and the region is further limited to the width and height of `mask`.

```c
void vgMask(VGImage mask,
            VGMaskOperation operation,
            VGint x, VGint y, VGint width, VGint height);
```

---

##   Mask operations

The `VGMaskOperation` enumeration defines the set of possible operations that may be used to modify a mask.

 - `VG_CLEAR_MASK` operation sets all mask values in the region of interest to `0`, ignoring the new mask image.

 - `VG_FILL_MASK` operation sets all mask values in the region of interest to `1`, ignoring the new mask image.

 - `VG_SET_MASK` operation copies values in the region of interest from the new mask image, overwriting the previous mask values.

 - `VG_UNION_MASK` operation replaces the previous mask in the region of interest by its union with the new mask image (the resulting values are always greater than or equal to their previous value).

 - `VG_INTERSECT_MASK` operation replaces the previous mask in the region of interest by its intersection with the new mask image (the resulting mask values are always less than or equal to their previous value).

 - `VG_SUBTRACT_MASK` operation subtracts the new mask from the previous mask and replaces the previous mask in the region of interest by the resulting mask (the resulting values are always less than or equal to their previous value).

The following table gives the equations defining the new mask value `α'` for each mask operation in terms of the previous mask value `α` and the newly supplied mask value `αMask`:

| Operation | Mask equation |
| :-------- | :------------ |
| `VG_CLEAR_MASK` | `α' = 0` |
| `VG_FILL_MASK` | `α' = 1` |
| `VG_SET_MASK` | `α' = αMask` |
| `VG_UNION_MASK` | `α' = 1 – (1 – αMask) * (1 – α)` |
| `VG_INTERSECT_MASK` | `α' = αMask * α` |
| `VG_SUBTRACT_MASK` | `α' = α * (1 – αMask)` |
{:.rwd-table}

## vgRenderToMask

The `vgRenderToMask` function modifies the current surface mask by applying the given `operation` to the set of coverage values associated with the rendering of the given `path`. If `paintModes` contains `VG_FILL_PATH`, the path is filled; if it contains `VG_STROKE_PATH`, the path is stroked. If both are present, the mask operation is performed in two passes, first on the filled path geometry, then on the stroked path geometry.

Conceptually, for each pass, an intermediate single-channel image is initialized to `0`, then filled with those coverage values that would result from the first four stages of the OpenVG pipeline (state setup, stroked path generation if applicable, transformation, and rasterization) when drawing a path with `vgDrawPath` using the given set of paint modes and all current OpenVG state settings that affect path rendering (scissor rectangles, fill rule, stroke parameters, etc.); paint settings are ignored. Finally, the drawing surface mask is modified as though `vgMask` were called using the intermediate image as the `mask` parameter. If operation is `VG_CLEAR_MASK` or `VG_FILL_MASK`, `path` is ignored and the entire mask is affected.

```c
void vgRenderToMask(VGPath path, VGbitfield paintModes, VGMaskOperation operation);
```

## The tutorial code
The tutorial draws a single path, representing a vector ship, with masking enabled; the ship data is defined in a separate file (see `ship.c` / `Ship.java`).

```c
VGPaint solidCol;
VGPath shipBackground;
VGPath ship;

// clear the whole drawing surface
vgClear(0, 0, surfaceWidth, surfaceHeight);
// enable masking
vgSeti(VG_MASKING, VG_TRUE);
// blue background
vgSetColor(solidCol, 0x3F6FBFFF);
vgSeti(VG_MATRIX_MODE, VG_MATRIX_PATH_USER_TO_SURFACE);
vgLoadIdentity();
vgDrawPath(shipBackground, VG_FILL_PATH);
// the yellow ship at the center of drawing surface
vgSetColor(solidCol, 0xFFBF0DFF);
vgTranslate(surfaceWidth / 2, surfaceHeight / 2);
vgDrawPath(ship, VG_FILL_PATH);
```

| ![the original vector ship, masking disabled]({{site.url}}/assets/images/tut10_ship.png) | 
| :---: |
| *the original vector ship, masking disabled* | 

Surface mask is generated by applying two *alpha primitives*; each alpha primitive can be chosen from a set of four available primitives:

 - a vector path representing a flat spot (defined in `mask_paths.c` / `MaskPaths.java`)

 - a vector path representing a tentacular spot (defined in `mask_paths.c` / `MaskPaths.java`)

 - a `512x512` bitmap image (`VG_A_8 forma`t), representing a smoke cloud (defined in `mask_images.c`)

 - a `512x512` bitmap image (`VG_A_8 format`), representing a burning star (defined in `mask_images.c`)

```c
typedef enum {
    // the "flat spot" path
    MaskPath0 = 0,
    // the "tentacular spot" path
    MaskPath1 = 1,
    // the "smoke cloud" image
    MaskImage0 = 2,
    // the "burning star" image
    MaskImage1 = 3
} AlphaPrimitiveType;
```

| ![The "flat spot" path]({{site.url}}/assets/images/tut10_spot1.png) | ![The "tentacular spot" path]({{site.url}}/assets/images/tut10_spot2.png) |
| :---: | :---: |
| *The "flat spot" path* | *The "tentacular spot" path* |

| ![The "smoke cloud" image]({{site.url}}/assets/images/tut10_cloud.png) | ![The "burning star" image]({{site.url}}/assets/images/tut10_star.png) |
| :---: | :---: |
| *The "smoke cloud" image* | *The "burning star" image* |

A first primitive is put on the surface mask using the `VG_SET_MASK` operation; a second alpha primitive is draw on top of the previous one using a variable mask operation (`VG_UNION_MASK`, `VG_INTERSECT_MASK`, `VG_SUBTRACT_MASK`).
The user can move both alpha primitives using mouse / touch, and at the same time can switch between mask operations, in order to experiment with them.

| ![VG_UNION_MASK operation]({{site.url}}/assets/images/tut10_union.png) | ![VG_INTERSECT_MASK operation]({{site.url}}/assets/images/tut10_intersection.png) | ![VG_SUBTRACT_MASK operation]({{site.url}}/assets/images/tut10_subtraction.png) |
| :---: | :---: | :---: |
| *VG_UNION_MASK operation* | *VG_INTERSECT_MASK operation* | *VG_SUBTRACT_MASK operation* |

```c
// first alpha primitive
AlphaPrimitiveType mask0Type;
float mask0Pos[2];
// second alpha primitive
AlphaPrimitiveType mask1Type;
float mask1Pos[2];
// current mask operation
VGMaskOperation maskOp;

// draw a single alpha mask primitive
void drawAlphaMaskPrimitive(const AlphaPrimitiveType type,
                            const float pos[],
                            const VGMaskOperation operation) {

    if ((type == MaskPath0) || (type == MaskPath1)) {
        // alpha path
        const VGPath mask = (type == MaskPath0) ? spotMask0 : spotMask1;
        vgRenderToMask(mask, VG_FILL_PATH, operation);
    }
    else {
        // alpha image
        const VGImage mask = (type == MaskImage0) ? starMaskImage : cloudMaskImage;
        vgMask(mask, operation, pos[X], pos[Y], 512, 512);
    }
}

// update the whole alpha mask (i.e. draw the current two alpha primitives)
void drawAlphaMask(void) {

    // draw first alpha primitive
    drawAlphaMaskPrimitive(mask0Type, mask0Pos, VG_SET_MASK);
    
    // draw second alpha primitive, on top of the previous one
    drawAlphaMaskPrimitive(mask1Type, mask1Pos, maskOp);
}
```

| ![Cloud image "intersected" with spot path]({{site.url}}/assets/images/tut10_cloud_spot_intersection.png) | ![Cloud image "subtracted" from the star image]({{site.url}}/assets/images/tut10_cloud_star_subtraction.png) |
| :---: | :---: |
| *Cloud image "intersected" with spot path* | *Cloud image "subtracted" from the star image* |

---