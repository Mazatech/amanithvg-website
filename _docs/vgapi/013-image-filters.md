---
layout: default
title: "Image filters"
date: 2018-11-01 08:00:00 +0100
chapter: 13
categories: [vgapi]
headline: "Openvg API, image filters definition"
image: "amanithvg-logo.png"
keywords: "openvg api image filters filter"
---

# Image Filters

Image filters allow images to be modified and/or combined using a variety of imaging operations. The destination area to be written is the intersection of the source and destination image areas. The entire source image area is used as the filter input. The source and destination images involved in the filter operation must not overlap (i.e., have any pixels in common within any common ancestor image).

---

## Format Normalization [12.1]

Source pixels are converted to one of `sRGBA`, `sRGBA_PRE`, `lRGBA`, or `lRGBA_PRE` formats, as determined by the current values of the `VG_FILTER_FORMAT_PREMULTIPLIED` and `VG_FILTER_FORMAT_LINEAR` parameters. Filtered pixels are then converted into the destination format using the normal pixel format conversion rules described in [3.4].

---

## Channel Masks [12.2]

The `VG_FILTER_CHANNEL_MASK` parameter specifies which destination channels are to be written. The parameter is supplied as a bitwise combination of values from the `VGImageChannel` enumeration (i.e. any combination of `VG_RED`, `VG_GREEN`, `VG_BLUE`, `VG_ALPHA`).

---

## Color Combination [12.3]

```c
void vgColorMatrix(VGImage dst, VGImage src, const VGfloat* matrix)
```

Computes a linear combination (i.e. 4x4 color multiplication) of color and alpha values from the normalized source image at each pixel.

---

## Convolution [12.4]

```c
void vgConvolve(VGImage dst, VGImage src, 
                VGint kernelW, VGint KernelH,
                VGint shiftX, VGint shiftY,
                const VGshort* kernel,
                VGfloat scale, VGfloat bias,
                VGTilingMode tilingMode)
```

Apply a user-supplied convolution kernel to a normalized source image.
The dimensions of the kernel are given by `kernelWidth` and `kernelHeight`; the kernel values are specified as `kernelWidth * kernelHeight` `VGshorts` in column-major order.
The `shiftX` and `shiftY` parameters specify a translation between the source and destination images.
The result of the convolution is multiplied by a `scale` factor, and a `bias` is added.

---
{:.hrlight}

```c
void vgSeparableConvolve(VGImage dst, VGImage src,
                         VGint kernelWidth, VGint kernelHeight,
                         VGint shiftX, VGint shiftY,
                         const VGshort* kernelX,
                         const VGshort* kernelY,
                         VGfloat scale, VGfloat bias,
                         VGTilingMode tilingMode)
```

Apply a user-supplied separable convolution kernel to a normalized source image.
A separable kernel is a two-dimensional kernel in which each entry kij is equal to a product kxi * kyj of elements from two one-dimensional kernels, one horizontal and one vertical.
The lengths of the one-dimensional arrays `kernelX` and `kernelY` are given by `kernelWidth` and `kernelHeight`, respectively; the kernel values are specified as arrays of `VGshorts`.
The `shiftX` and `shiftY` parameters specify a translation between the source and destination images. The result of the convolution is multiplied by a `scale` factor, and a `bias` is added.

---
{:.hrlight}

```c
void vgGaussianBlur(VGImage dst, VGImage src,
                    VGfloat stdDevX, VGfloat stdDevY,
                    VGTilingMode tilingMode)
```

Compute the convolution of a normalized source image with a separable kernel defined in each dimension by the Gaussian function. 
Source pixels outside the source image bounds are defined by `tilingMode`, which takes a value from the `VGTilingMode` enumeration [9.4.1] 
The operation is applied to all channels (color and alpha) independently.

---

## Convolution Parameters

Read-only `paramType` values for the `vgGetParameter` function.

| Parameter name | Parameter type | Description |
| -------------- | -------------- | ----------- |
| `VG_MAX_KERNEL_SIZE` | `VGint` | The largest legal value of the width and height parameters to the `vgConvolve` function.<br>All implementations must define `VG_MAX_KERNEL_SIZE` to be an integer no smaller than `7` |
| `VG_MAX_SEPARABLE_KERNEL_SIZE` | `VGint` | The largest legal value of the size parameter to the `vgSeparableConvolve` function.<br>All implementations must define this value to be an integer no smaller than `15` |
| `VG_MAX_GAUSSIAN_STD_DEVIATION` | `VGint` | The largest legal value of the `stdDeviationX` and `stdDeviationY` parameters to the `vgGaussianBlur` function.<br>All implementations must define this value to be an integer no smaller than `16`.
{:.rwd-table .rwd-tableConvolutionParameters}

---

## Lookup Table [12.5]

```c
void vgLookup(VGImage dst, VGImage src,
              const VGubyte* redLUT,
              const VGubyte* greenLUT,
              const VGubyte* blueLUT,
              const VGubyte* alphaLUT,
              VGboolean outputLinear,
              VGboolean outputPremultiplied)
```

Pass each image channel of the normalized source image through a separate lookup table.
Each channel of the normalized source pixel is used as an index into the lookup table for that channel by multiplying the normalized value by `255` and rounding to obtain an 8-bit integral value.
Each `LUT` parameter should contain 256 `VGubyte` entries.

---
{:.hrlight}

```c
void vgLookupSingle(VGImage dst, VGImage src,
                    const VGuint* LUT, 
                    VGImageChannel sourceChannel, 
                    VGboolean outputLinear, 
                    VGboolean outputPremultiplied)
```

Pass a single image channel of the normalized source image, selected by the `sourceChannel` parameter, through a combined lookup table that produces whole pixel values.
Each normalized source channel value is multiplied by `255` and rounded to obtain an 8-bit integral value.
The specified `sourceChannel` of the normalized source pixel is used as an index into the lookup table.
The `lookupTable` parameter should contain 256 4-byte aligned entries

---
