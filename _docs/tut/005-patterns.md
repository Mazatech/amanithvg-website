---
layout: default
title: "Tut. 05 - Patterns"
date: 2018-11-01 08:00:00 +0100
chapter: 5
categories: [tut]
headline: "AmanithVG, how to create and use conical pattern paint type"
image: "tut05_repeat_dark.png"
keywords: "amanithvg tutorial 05 pattern fill pad repeat reflect openvg"
---

# Tutorial 05: Patterns

This tutorial will cover the last paint type available for filling and/or stroking paths: patterns.
Patterns are based on bitmap images, so before to start with the real tutorial, we have to introduce images as they are defined by OpenVG specifications.

---

## Images in OpenVG

Images are rectangular collections of pixels. Image data may be inserted or extracted in a variety of formats with varying bit depths, color spaces, and alpha channel types.
Images may be drawn to a drawing surface, used to define paint patterns, or operated on directly by image filter operations.

An image defines a coordinate system in which pixels are indexed using integer coordinates, with each integer corresponding to a distinct pixel. The lower-left pixel has a coordinate of `(0, 0)`, the x coordinate increases horizontally from left to right, and the y coordinate increases vertically from bottom to top.

The `VGImageFormat` enumeration defines the set of supported pixel formats and color spaces for images.
The letter `A` denotes an alpha channel, `R` denotes red, `G` denotes green, and `B` denotes blue. `X` denotes a padding byte that is ignored. `L` denotes grayscale, and `BW` denotes (linear) bi-level grayscale (black-and-white), with `0` representing black and `1` representing white in either case. A lower-case letter `s` represents a non-linear, perceptually-uniform color space, as in `sRGB` and `sL`; a lower-case letter `l` represents a linear color space using the `sRGB` primaries. Formats with a suffix of `_PRE` store pixel values in premultiplied format.

| Format | Bytes per pixel | Bits per pixel |
| ------ | --------------- | -------------- |
| `VG_sRGBX_8888` | 4 | 32 |
| `VG_sRGBA_8888` | 4 | 32 |
| `VG_sRGBA_8888_PRE` | 4 | 32 |
| `VG_sRGB_565` | 2 | 16 |
| `VG_sRGBA_5551` | 2 | 16 |
| `VG_sRGBA_4444` | 2 | 16 |
| `VG_sL_8` | 1 | 8 |
| `VG_lRGBX_8888` | 4 | 32 |
| `VG_lRGBA_8888` | 4 | 32 |
| `VG_lRGBA_8888_PRE` | 4 | 32 |
| `VG_lL_8` | 1 | 8 |
| `VG_A_1` | n/a | 1 |
| `VG_A_4` | n/a | 4 |
| `VG_A_8` | 1 | 8 |
| `VG_BW_1` | n/a | 1 |
{:.rwd-table .rwd-tableImagesOpenvg}

Other available byteorder formats (`A/XRGB`, `BGRA/X`, `A/XBGR`) follow the same "bytes/bits per pixel" rules.

Images can be created through the `vgCreateImage` function, specifying format and dimensions (in pixels).
The `vgImageSubData` function reads pixel values from memory, performs format conversion if necessary, and stores the resulting pixels into a rectangular portion of a given image; it's the core function used to upload image pixels to the OpenVG backend (as a simplification, it could be though as the OpenVG counterpart of the OpenGL `glTexSubImage2D` / `glTextureSubImage2D`). For a more comprehensive overview, take a look at [images section]({{site.url}}/docs/vgapi/011-images.html).

---

## Patterns in OpenVG

Pattern paint defines a rectangular pattern of colors based on the pixel values of an image.

Pattern paints have a reference system that coincides with images: the lower-left pixel has a coordinate of `(0, 0)`, the x coordinate increases horizontally from left to right, and the y coordinate increases vertically from bottom to top. Such system is then transported to the path coordinates system throught a "paint-to-user" affine matrix (`VG_MATRIX_FILL_PAINT_TO_USER` / `VG_MATRIX_STROKE_PAINT_TO_USER`). Finally the filled/stroked path is moved to the drawing surface system by another affine matrix, the so called "path-user-to-surface" (`VG_MATRIX_PATH_USER_TO_SURFACE`).

| &nbsp; |
| :---: |
| *Pattern Matrices* |
{:.tbl_images .tut05_pattern_matrices} 

The tutorial defines a simple procedural image, then creates a pattern paint object and link the image to it (see `genPaints` function); here's a simplified code version for a fixed 64x64 pattern image:

```c
unsigned int pixels[64 * 64], x, y;
unsigned int colors[16] = {
    0xFF6030FF, 0xFFB060FF, 0xFF9090FF, 0xFF30B0FF,
    0x60FF30FF, 0xB0FF60FF, 0x90FF90FF, 0x30FFB0FF,
    0x6030FFFF, 0xB060FFFF, 0x9090FFFF, 0x30B0FFFF,
    0x303030FF, 0x606060FF, 0x909090FF, 0xB0B0B0FF
};

// create pattern image
VGImage image = vgCreateImage(VG_sRGBA_8888_PRE, 64, 64, VG_IMAGE_QUALITY_BETTER);

// generate procedural pixels (chessboard-like)
for (y = 0; y < 64; ++y) {
    unsigned int i = y / 16;
    for (x = 0; x < 64; ++x) {
        unsigned int j = x / 16;
        pixels[(y * 64) + x] = colors[(i * 4) + j];
    }
}

// upload pixels to the OpenVG backend
vgImageSubData(image, pixels, 64 * sizeof(unsigned int),
               VG_sRGBA_8888_PRE, 0, 0, 64, 64);

// create paint object
VGPaint pattern = vgCreatePaint();
// set the paint object to be a pattern paint
vgSetParameteri(pattern, VG_PAINT_TYPE, VG_PAINT_TYPE_PATTERN);
// link the image to the pattern paint object
vgPaintPattern(pattern, image);
```

| &nbps; |
| :---: |
| *64x64 pattern image* |
{:.tbl_images .tut05_pattern_image} 

As it can be seen from the code, to enable pattern paints, we use `vgSetParameteri` to set the paint type to `VG_PAINT_TYPE_PATTERN`.
The `vgPaintPattern` function, instead, replaces any previous pattern image defined on the given paint object with a new pattern image.

---

## Pattern tiling

Patterns may be extended (tiled) using one of four possible tiling modes, defined by the `VGTilingMode` enumeration:

 - the `VG_TILE_FILL` condition specifies that pixels outside the bounds of the source image should be taken as the color `VG_TILE_FILL_COLOR`. Such color is expressed as a non-premultiplied `sRGBA` color and alpha value. Values outside the `[0, 1]` range are interpreted as the nearest endpoint of the range.

| &nbsp; |
| :---: |
| *Tile fill* |
{:.tbl_images .tut05_tile_fill} 

 - the `VG_TILE_PAD` condition specifies that pixels outside the bounds of the source image should be taken as having the same color as the closest edge pixel of the source image; that is, a pixel `(x, y)` has the same value as the image pixel `(max(0, min(x, width – 1))`, `max(0, min(y, height – 1)))`.

| &nbsp; |
| :---: |
| *Tile pad* |
{:.tbl_images .tut05_tile_pad} 

 - the `VG_TILE_REPEAT` condition specifies that the source image should be repeated indefinitely in all directions; that is, a pixel `(x, y)` has the same value as the image pixel (`x mod width`, `y mod height`) where the operator `(a mod b)` returns a value between `0` and `(b – 1)` such that `a = (k * b) + (a mod b)` for some integer `k`. For example, a such module operator coincides with the standard C/C++/Java `%` operator.

| &nbsp; |
| :---: |
| *Tile repeat* |
{:.tbl_images .tut05_tile_repeat} 

 - the `VG_TILE_REFLECT` condition specifies that the source image should be reflected indefinitely in all directions. That is, a pixel `(x, y)` has the same value as the image pixel `(u, v)` where:
   
   - `u = (x mod width)` if `floor(x/width)` is even; `(width – 1 – (x mod width))` otherwise
   
   - `v = (y mod height)` if `floor(y/height)` is even; `(height – 1 – (y mod height))` otherwise

| &nbsp; |
| :---: |
| *Tile reflect* |
{:.tbl_images .tut05_tile_reflect} 

The pattern tiling mode is set using `vgSetParameteri` with a `paramType` argument of `VG_PAINT_PATTERN_TILING_MODE`.

```c
VGfloat tileColor[4] = { 0.1f, 0.6f, 0.3f, 1.0f };
// set tiling mode
vgSetParameteri(pattern, VG_PAINT_PATTERN_TILING_MODE, VG_TILE_FILL);
vgSetfv(VG_TILE_FILL_COLOR, 4, tileColor);
```

--- 

## The tutorial code 

The tutorial draws a circle path at the center of drawing surface, filled with a colored pattern.
The path is created in object space with center at `(0, 0)` and a radius of `1` (see `genPaths` function):

```c
// create the circle that will be filled by the radial gradient paint
filledCircle = vgCreatePath(VG_PATH_FORMAT_STANDARD, VG_PATH_DATATYPE_F,
                            1.0f, 0.0f, 0, 0, VG_PATH_CAPABILITY_ALL);
vguEllipse(filledCircle, 0.0f, 0.0f, 2.0f, 2.0f);
```

By having defined the path in this tricky way, it's really easy to scale and center it in order to fit the drawing surface:

```c
// find the minimum dimension between surface width and height, then halve it
int halfDim = (surfaceWidth < surfaceHeight) ? (surfaceWidth / 2) : (surfaceHeight / 2);
// calculate scale factor in order to cover 90% of it
userToSurfaceScale = halfDim * 0.9f;
// translate to the surface center
userToSurfaceTranslation[X] = surfaceWidth / 2;
userToSurfaceTranslation[Y] = surfaceHeight / 2;

// "user to surface" transformation (a uniform scale plus a translation),
// upload the matrix to the OpenVG backend
vgSeti(VG_MATRIX_MODE, VG_MATRIX_PATH_USER_TO_SURFACE);
vgLoadIdentity();
vgTranslate(userToSurfaceTranslation[X], userToSurfaceTranslation[Y]);
vgScale(userToSurfaceScale, userToSurfaceScale);
```

The user can set the pattern position and orientation through two control points, called `center` and `target` (defined in surface space).
Such control points are highlighted by two white spots that can be moved by using mouse/touch.

In order to accomplish the task to map the pattern image to the desired bounds, we have to review some details (see `setMatrices` function):

 - we define the direction vector (i.e. the baseline where we want to lay the image) in surface space as: `direction = target - center`

 - the rotation factor is given by the relative slope between the two control points: `rotation = arctan(direction[Y] / direction[X])`

 - the scale factor is given by the distance between the two control points divided by the image size (e.g. 64 in the example above): `scale = length(direction) / imageSize`

 - the translation factor is given by the `center` point

```c
// calculate pattern direction
float direction[] = {
    patternTarget[X] - patternCenter[X],
    patternTarget[Y] - patternCenter[Y]
};
// calculate scale factor
float dist = hypot(direction[X], direction[Y]);
float scale = dist / patternImageSize;
// calculate rotation factor
float rotation = atan2(direction[Y], direction[X]);

// "paint to user" transformation, upload the matrix to the OpenVG backend
vgSeti(VG_MATRIX_MODE, VG_MATRIX_FILL_PAINT_TO_USER);
vgLoadIdentity();
vgTranslate(patternCenter[X], patternCenter[Y]);
vgScale(scale, scale);
// OpenVG rotations must be expressed in degrees
vgRotate(rotation * 57.2957f);
```

The code represents the mapping between the pattern coordinates system and the drawing surface system (where control points live).
But we know that, in between, there is the "user-to-surface" transformation too, which unfortunately interferes with the desired mapping.
So, by setting the `VG_MATRIX_FILL_PAINT_TO_USER` matrix, we must take care to nullify the effect of `VG_MATRIX_PATH_USER_TO_SURFACE` matrix by appending its inverse transformation (i.e. essentially dividing `scale` and `patternCenter` values by `userToSurfaceScale`; see `setMatrices` function for the code details).

---
