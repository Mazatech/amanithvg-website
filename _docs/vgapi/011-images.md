---
layout: default
title: "Images"
date: 2017-01-01 08:00:00 +0100
chapter: 11
categories: [vgapi]
---

# Images

Images are rectangular collections of pixels. Image data may be inserted or extracted in a variety of formats with varying bit depths, color spaces, and alpha channel types. The actual storage format of an image is implementation-dependent, and may be optimized for a given device, but must allow pixels to be read and written losslessly. Images may be drawn to a drawing surface, used to define paint patterns, or operated on directly by image filter operations.

---

## Image Coordinate Systems [10.1]

An image defines a coordinate system in which pixels are indexed using integer coordinates, with each integer corresponding to a distinct pixel. The lower-left pixel has a coordinate of (0, 0), the x coordinate increases horizontally from left to right, and the y coordinate increases vertically from bottom to top.

---

## Image Definition [10.3]

Images are accessed using opaque handles of type `VGImage`.

```c
typedef VGHandle VGImage;
```

## Create & Destroy Image [10.3]

```c
VGImage vgCreateImage(VGImageFormat fmt,
                      VGint width, VGint height,
                      VGbitfield quality)
```

Create an image with the given `width`, `height`, and pixel format and returns a `VGImage` handle to it.  
The format parameter must contain a value from the `VGImageFormat` enumeration.  
The `allowedQuality` parameter is a bitwise OR of values from the `VGImageQuality` enumeration, indicating which levels of resampling quality may be used to draw the image.

---

```c
void vgDestroyImage(VGImage image)
```

Deallocate an image, releasing any resources associated with it; following the call, the image handle is no longer valid in any context that shared it.  
If the image is currently in use as a rendering target, is the ancestor of another image, is set as a paint pattern image on a `VGPaint` object, or is set as a glyph an a `VGFont` object, its definition remains available to those consumers as long as they remain valid, but the handle may no longer be used.  
When those uses cease, the image's resources will automatically be deallocated.

---

## Image Object Parameter [10.4]

Values from the `VGImageParamType` enumeration may be used as the paramType argument to vgGetParameter to query various features of an image. All of the parameters defined by `VGImageParamType` have integer values and are read-only.

| Parameter name | Parameter type | Possible values | Notes |
| -------------- | -------------- | --------------- | ----- |
| `VG_IMAGE_FORMAT` | VGImageFormat| `// RGB{A,X}` channel ordering<br> `VG_sRGB_565`<br> `VG_{s,l}RGBX_8888`<br> `VG_{s,l}RGBA_PRE`<br> `VG_sRGBA_{5551,4444}`<br> `VG_{sL,lL,A}_8`<br> `VG_{BW,A}_1`<br> `// {A,X}RGB` channel ordering<br> `VG_{s,l}{XRGB,ARGB}_8888`<br> `VG_{s,l}ARGB_8888_PRE`<br> `VG_{sARGB}_{1555,4444}`<br> `// {BGR{A,X}` channel ordering<br> `VG_{s,l}{BGRX,BGRA}_8888`<br> `VG_{s,l}BGRA_8888_PRE`<br> `VG_{sBGRA}_{1555,4444}`<br> `// {A,X}BGR` channel ordering<br> `VG_{s,l}{XBGR,ABGR}_8888`<br> `VG_{s,l}ABGR_8888_PRE`<br> `VG_{sABGR}_{1555,4444}`<br> `VG_sRGB_565` | Specified through the `vgCreateImage` function |
| `VG_IMAGE_WIDTH` | `VGint` | Any value between `0` and `VG_MAX_IMAGE_WIDTH` | Specified through the `vgCreateImage` function |
| `VG_IMAGE_HEIGHT` | `VGint` | Any value between `0` and `VG_MAX_IMAGE_HEIGHT` | Specified through the vgCreateImage function |
{:.rwd-table}

---

## Read and Write Image Pixels [10.5]

```c
void vgClearImage(VGImage image,
                  VGint x, VGint y,
                  VGint width, VGint height)
```

Fill a given rectangle of an image with the color specified by the `VG_CLEAR_COLOR` parameter.  
The rectangle to be cleared is given by `x`, `y`, `width`, and `height`, which must define a positive region.  
The rectangle is clipped to the bounds of the image.

---
{:.hrlight}

```c
void vgImageSubData(VGImage image,
                    const void* data,
                    VGint dataStride, 
                    VGImageFormat fmt,
                    VGint x, VGint y, 
                    VGint width, VGint height)
```

Read pixel values from memory, perform format conversion if necessary, and store the resulting pixels into a rectangular portion of an image.

---
{:.hrlight}

```c
void vgGetImageSubData(VGImage image,
                       void* data,
                       VGint dataStride,
                       VGImageFormat fmt,
                       VGint x, VGint y,
                       VGint width, VGint height)
```

Read pixel values from a rectangular portion of an image, perform format conversion if necessary, and store the resulting pixels into memory.

---

## Child Images [10.6]

A child image is an image that shares physical storage with a portion of an existing image, known as its parent. An image may have any number of children, but each image has only one parent (that may be itself). Changes to an image are immediately reflected in all other images to which it is related. A child image may not be used as a rendering target.

```c
VGImage vgChildImage(VGImage parent,
                     VGint x, VGint y,
                     VGint width, VGint height)
```

Return a new `VGImage` handle that refers to a portion of the parent image.
The region is given by the intersection of the bounds of the parent image with the rectangle beginning at pixel `(x, y)` with dimensions `width` and `height`, which must define a positive region contained entirely within parent.

---
{:.hrlight}

```c
VGImage vgGetParent(VGImage image)
```

Return the closest valid ancestor of the given image. If image has no ancestors, image is returned.

---

## Copy Between Images [10.7]

```c
void vgCopyImage(VGImage dst,
                 VGint dx, VGint dy,
                 VGImage src,
                 VGint sx, VGint sy,
                 VGint width, VGint height,
                 VGboolean dither)
```

Pixels may be copied between images using the vgCopyImage function.  
The source image pixel `(sx + i, sy + j)` is copied to the destination image pixel `(dx + i, dy + j)`, `for 0 <= i < width` and `0 <= j < height`.  
Pixels whose source or destination lie outside of the bounds of the respective image are ignored.  
Pixel format conversion is applied as needed.

---

## Draw Image [10.8]

Images may be drawn onto a drawing surface. An affine or projective transformation may be applied while drawing. The current image and blending modes are used to control how image pixels are combined with the current paint and blended into the destination.

```c
void vgDrawImage(VGImage image)
```

Drawn the given image to the current drawing surface; the current image-user-to-surface transformation (`VG_MATRIX_IMAGE_USER_TO_SURFACE`) is applied to the image.  
When a projective transformation is used, the value of the `VG_IMAGE_MODE` parameter is ignored and the behavior of `VG_DRAW_IMAGE_NORMAL` is substituted.

---

## Read and Write Drawing Surface Pixels [10.9]

```c
void vgSetPixels(VGint dx, VGint dy,
                 VGImage src,
                 VGint sx, VGint sy,
                 VGint width, VGint height)
```

Copy pixel data from the image `src` onto the drawing surface.  
The image pixel `(sx + i, sy + j)` is copied to the drawing surface pixel `(dx + i, dy + j)`, `for 0 <= i < width` and `0 <= j < height`.  
Pixels whose source lies outside of the bounds of src or whose destination lies outside the bounds of the drawing surface are ignored. Scissoring takes place normally.  
Transformations, masking, and blending are not applied.

---
{:.hrlight}

```c
void vgWritePixels(const void* data,
                   VGint dataStride,
                   VGImageFormat fmt,
                   VGint dx, VGint dy,
                   VGint width, VGint height)
```

Copy provided pixel data to the drawing surface without the creation of a `VGImage` object.  
The pixel values to be drawn are taken from the data pointer at the time of the `vgWritePixels` call, so future changes to the data have no effect.  
Pixels whose destination coordinate lies outside the bounds of the drawing surface are ignored.  
Scissoring takes place normally.  
Transformations, masking, and blending are not applied.

---
{:.hrlight}

```c
void vgGetPixels(VGImage dst,
                 VGint dx, VGint dy,
                 VGint sx, VGint sy,
                 VGint width, VGint height)
```

Retrieve pixel data from the drawing surface into the image `dst`.  
The drawing surface pixel `(sx + i, sy + j)` is copied to pixel `(dx + i, dy + j)` of the image `dst`, `for 0 <= i < width` and `0 <= j < height`.  
Pixels whose source lies outside of the bounds of the drawing surface or whose destination lies outside the bounds of dst are ignored.  
The scissoring region does not affect the reading of pixels.

---
{:.hrlight}

```c
void vgReadPixels(void* data,
                  VGint dataStride,
                  VGImageFormat fmt,
                  VGint sx, VGint sy,
                  VGint width, VGint height)
```

Copy pixel data from the drawing surface without the creation of a `VGImage` object.  
Pixels whose source lies outside of the bounds of the drawing surface are ignored.  
Pixel format conversion is applied as needed.  
The scissoring region does not affect the reading of pixels.

---
{:.hrlight}

```c
void vgCopyPixel(VGint dx, VGint dy,
                 VGint sx, VGint sy,
                 VGint width, VGint height)
```

Copy pixels from one region of the drawing surface to another.  
The drawing surface pixel `(sx + i, sy + j)` is copied to pixel `(dx + i, dy + j)` `for 0 <= i < width` and `0 <= j < height`.  
Pixels whose source or destination lies outside of the bounds of the drawing surface are ignored. Transformations, masking, and blending are not applied.  
Scissoring is applied to the destination, but does not affect the reading of pixels.

---

## Pixel Copy Functions [10.9]

| Src/Dst | Memory | VGImage | Surface |
| ------- | ------ | ------- | ------- |
| `Memory` | `-` | `vgImageSubData` | `vgWritePixels` |
| `VGImage` | `vgGetImageSubData` | `vgCopyImage` | `vgSetPixels` |
| `Surface` | `vgReadPixels` | `vgGetPixels` | `vgCopyPixels` |
{:.rwd-table}

---
