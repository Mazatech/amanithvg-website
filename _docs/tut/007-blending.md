---
layout: default
title: "Tut. 07 - Alpha blending"
date: 2017-01-01 08:00:00 +0100
chapter: 7
categories: [tut]
headline: "AmanithVG, how to use alpha blending operation"
image: "tut07_src_over.png"
keywords: "amanithvg tutorial 07 path alpha blending Porter-Duff src src-over openvg"
---

# Tutorial 07: Alpha blending

According to OpenVG specifications, the pipeline mechanism by which primitives are rendered consists in the following steps:

 - **path/image transformation** to the drawing surface reference system (the current path-user-to-surface transformation is applied to the path geometry, producing drawing surface coordinates; for an image, the outline of the image is transformed using the image-user-to-surface transformation)

 - **rasterization** (a coverage value is computed at pixels affected by the drawn path/image)

 - **clipping and masking** (pixels not lying within the bounds of the drawing surface, or not lying within the union of the current set of active scissor rectangles, are assigned a coverage value of 0)

 - **paint generation** (at each pixel of the drawing surface, the relevant current paint is used to define a color and an alpha value)

 - **blending** (at each pixel, the source color and alpha value from the preceding stage is converted into the destination color space; the resulting color is blended with the corresponding destination color and alpha value according to the current blending rule)

Previous tutorials have introduced all the pipeline stages except the last one: now it's time to take a look at *blending*.

---

## Blending equations

A blending mode defines an alpha blending function `α(Asrc, Adst)` and a color blending function `c(RGBsrc, RGBdst, Asrc, Adst)`. Given a non-premultiplied source color and alpha tuple `(Rsrc, Gsrc, Bsrc, Asrc)` and a non-premultiplied destination color and alpha tuple `(Rdst, Gdst, Bdst, Adst)`, blending replaces the destination with the blended tuple `(c(Rsrc, Rdst, Asrc, Adst)`, `c(Gsrc, Gdst, Asrc, Adst)`, `c(Bsrc, Bdst, Asrc, Adst)`, `α(Asrc, Adst))`.

If either the source or destination is stored in a premultiplied format (i.e., pixels are stored as tuples of the form `(A * R, A * G, A * B, A)`), the alpha value is conceptually divided out prior to applying the blending equations described above. If the destination is premultiplied, the destination color values are clamped to the range `[0, A]` when read, and the destination alpha value is multiplied into each color channel prior to storage. If the destination format does not store alpha values, an alpha value of `1` is used in place of `Adst`.

---

## Porter-Duff blending

Given two premultiplied colors and their relative alpha values, Porter-Duff blending defines the resulting (premultiplied) color and alpha value by the following general equation:

`Dca' = f(Sc, Dc) * Sa * Da + Y * Sca * (1 - Da) + Z * Dca * (1 - Sa)`

`Da'  =         X * Sa * Da + Y *  Sa * (1 - Da) + Z *  Da * (1 - Sa)`

where:

 - `Sc` is the non-premultiplied source color component `(R, G, B triplet)`
 
 - `Sa` is the source opacity component

 - `Sca` is the premultiplied source color component `(A * R, A * G, A * B triplet)`

 - `Dc` is the non-premultiplied destination color component `(R, G, B triplet)`

 - `Da` is the destination opacity component

 - `Dca` is the premultiplied destination color component `(A * R, A * G, A * B triplet)`

 - `f(Sc, Dc)` is the intersection of the opacity of the source and destination multiplied by some function of the color (used for color)

 - `X` is the intersection of the opacity of the source and destination (used for opacity)

 - `Y` is the intersection of the source and the inverse of the destination

 - `Z` is the intersection of the inverse of the source and the destination

Depending on the compositing operation, each of the above values may or may not be used in the generation of the destination pixel value.

---

## Blending in OpenVG

The `VGBlendMode` enumeration defines the possible blending modes: `VG_BLEND_SRC`, `VG_BLEND_SRC_OVER`, `VG_BLEND_DST_OVER`, `VG_BLEND_SRC_IN`, `VG_BLEND_DST_IN`, `VG_BLEND_MULTIPLY`, `VG_BLEND_SCREEN`, `VG_BLEND_DARKEN`, `VG_BLEND_LIGHTEN`, `VG_BLEND_ADDITIVE`.
We can use `vgSeti` function, with a parameter type of `VG_BLEND_MODE`, to set the blend mode.

```c
vgSeti(VG_BLEND_MODE, VG_BLEND_SRC_OVER);
```

The following table shows the equation factors for each OpenVG blend mode:

| Blend mode | f(Sc, Dc) |  X  |  Y  |  Z  |
| :--------- | :-------- | :-: | :-: | :-: |
| `VG_BLEND_SRC` | `Sc` | `1` | `1` | `0` |
| `VG_BLEND_SRC_OVER` | `Sc` | `1` | `1` | `1` |
| `VG_BLEND_DST_OVER` | `Dc` | `1` | `1` | `1` |
| `VG_BLEND_SRC_IN` | `Sc` | `1` | `0` | `0` |
| `VG_BLEND_DST_IN` | `Dc` | `1` | `0` | `0` |
| `VG_BLEND_MULTIPLY` | `ScDc` | `1` | `1` | `1` |
| `VG_BLEND_SCREEN` | `Sc + Dc(1 - Sc)` | `1` | `1` | `1` |
| `VG_BLEND_DARKEN` | `min(Sc, Dc)` | `1` | `1` | `1` |
| `VG_BLEND_LIGHTEN` | `max(Sc, Dc)` | `1` | `1` | `1` |
| `VG_BLEND_ADDITIVE` | `Sc + Dc` | `1` | `1` | `1` | 
{:.rwd-table}

The following table shows the resulting (and simplified) blend equation for each OpenVG blend mode:

| Blend mode | Dca' |  Da'  |  
| :--------- | :--- | :---- |
| `VG_BLEND_SRC` | `Sca` | `Sa` |
| `VG_BLEND_SRC_OVER` | `Sca + Dca(1 - Sa)` | `Sa + Da(1 - Sa)` |
| `VG_BLEND_DST_OVER` | `Dca + Sca(1 - Da)` | `Da + Sa(1 - Da)` |
| `VG_BLEND_SRC_IN` | `ScaDa` | `SaDa` |
| `VG_BLEND_DST_IN` | `DcaSa` | `SaDa` |
| `VG_BLEND_MULTIPLY` | `ScaDca + Sca(1 - Da) + Dca(1 - Sa)` | `Sa + Da(1 - Sa)` |
| `VG_BLEND_SCREEN` | `Sca + Dca(1 - Sca)` | `Sa + Da(1 - Sa)` |
| `VG_BLEND_DARKEN` | `min(ScaDa, DcaSa) + Sca(1 - Da) + Dca(1 - Sa)` | `Sa + Da(1 - Sa)` |
| `VG_BLEND_LIGHTEN` | `max(ScaDa, DcaSa) + Sca(1 - Da) + Dca(1 - Sa)` | `Sa + Da(1 - Sa)` |
| `VG_BLEND_ADDITIVE` | `Sca + Dca` | `Sa + Da` |
{:.rwd-table}

---

## The tutorial code

The tutorial code is really simple. Two images are drawn on top of each other: first a *destination* image is drawn using the `VG_BLEND_SRC` blend mode, then a *source* image is drawn on top of the previous one using a variable blend mode.
The user can change the blend mode and, at the same time, can move both images and see the visual result of blending equations.

```c
// SRC and DST images
VGImage srcImage;
VGImage dstImage;
// the current blend mode
VGBlendMode blendMode;

// draw DST image
vgSeti(VG_BLEND_MODE, VG_BLEND_SRC);
vgSeti(VG_MATRIX_MODE, VG_MATRIX_IMAGE_USER_TO_SURFACE);
vgLoadIdentity();
vgTranslate(dstImagePos[X], dstImagePos[Y]);
vgDrawImage(dstImage);
// draw SRC image, setting the current blend mode
vgSeti(VG_BLEND_MODE, blendMode);
vgLoadIdentity();
vgTranslate(srcImagePos[X], srcImagePos[Y]);
vgDrawImage(srcImage);
```

| ![Source image "over" destination image (`VG_BLEND_SRC_OVER`)]({{site.url}}/assets/images/tut07_src_over.png) | 
| :---: |
| *Source image "over" destination image (`VG_BLEND_SRC_OVER`)* | 

Both images are generated procedurally according to the current surface size. A flower-like path is drawn (at the lower-left corner) with a [linear gradient]({{site.url}}/docs/tut/002-linear-gradients.html) fill, then pixels are copied from the drawing surface to the `VGImage` object, using the `vgGetPixels`function.

```c
// create images
srcImage = vgCreateImage(imgFormat, imgSize, imgSize, VG_IMAGE_QUALITY_NONANTIALIASED);
dstImage = vgCreateImage(imgFormat, imgSize, imgSize, VG_IMAGE_QUALITY_NONANTIALIASED);

// clear surface with a transparent black
vgSetfv(VG_CLEAR_COLOR, 4, black);
vgSeti(VG_BLEND_MODE, VG_BLEND_SRC);

// generate SRC image
vgClear(0, 0, surfaceWidth, surfaceHeight);
vgSetPaint(paintSrc, VG_FILL_PATH);
vgDrawPath(flower, VG_FILL_PATH);
vgGetPixels(srcImage, 0, 0, 0, 0, imgSize, imgSize);

// generate DST image
vgClear(0, 0, surfaceWidth, surfaceHeight);
vgSetPaint(paintDst, VG_FILL_PATH);
vgDrawPath(flower, VG_FILL_PATH);
vgGetPixels(dstImage, 0, 0, 0, 0, imgSize, imgSize);
```

| ![Source image]({{site.url}}/assets/images/tut07_src_image.png) | ![Destination image]({{site.url}}/assets/images/tut07_dst_image.png) |
| :---: | :---: |
| *Source image* | *Destination image* |

A note about the image format value (`VGImageFormat enum` type) used to create images: in order to speedup "read pixels" and rendering operations, the best practice is to create images using the same (internal) format of the drawing surface; such format could be retrieved through the AmanithVG function `vgPrivGetSurfaceFormatMZT` (see [EGL like interface]({{site.url}}/docs/desc/003-egl.html)).

---

## Blending extensions

AmanithVG implements an extension to the OpenVG API relative to blending: `VG_MZT_advanced_blend_modes` (see [extensions]({{site.url}}/docs/desc/004-extensions.html) chapter for additional details).
This extension completes the OpenVG blend modes in order to support a full [Porter-Duff alpha-compositing model](http://www.w3.org/TR/2009/WD-SVGCompositing-20090430/#comp-op). In the detail the extension introduces several new blend modes: `VG_BLEND_CLEAR_MZT`, `VG_BLEND_DST_MZT`, `VG_BLEND_SRC_OUT_MZT`, `VG_BLEND_DST_OUT_MZT`, `VG_BLEND_SRC_ATOP_MZT`, `VG_BLEND_DST_ATOP_MZT`, `VG_BLEND_XOR_MZT`, `VG_BLEND_OVERLAY_MZT`, `VG_BLEND_COLOR_DODGE_MZT`, `VG_BLEND_COLOR_BURN_MZT`, `VG_BLEND_HARD_LIGHT_MZT`, `VG_BLEND_SOFT_LIGHT_MZT`, `VG_BLEND_DIFFERENCE_MZT`, `VG_BLEND_EXCLUSION_MZT`.

| ![Destination image "at top" of source image (`VG_BLEND_DST_ATOP_MZT`)]({{site.url}}/assets/images/tut07_dst_atop.png) | 
| :---: |
| *Destination image "at top" of source image (`VG_BLEND_DST_ATOP_MZT`)* | 

The tutorial application will check (at runtime) the support of `VG_MZT_advanced_blend_modes` extension and, if found, will enable the user to switch to advanced blend modes too.

---