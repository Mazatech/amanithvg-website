---
layout: default
title: "Color transformation, blending"
date: 2017-01-01 08:00:00 +0100
chapter: 14
categories: [vgapi]
headline: "Openvg API, color transformation and blending definition"
image: "amanithvg-logo.png"
keywords: "openvg api color transformation blending equation transform blend premultiply unpremultiply"
---

# Color Transformation and Blending

In the final pipeline stage, the pixels from the previous pipeline stage (paint generation or image interpolation) are optionally transformed by a color transformation matrix. The resulting pixels are converted into the destination color space, and blending is
performed using a subset of the standard Porter-Duff blending rules along with several additional rules.

---

## Color Transformation [13.1]

If the `VG_COLOR_TRANSFORM` parameter is enabled, each color from the preceding pipeline stage is converted to non-premultiplied form. Each channel is multiplied by a per-channel `scale` factor, and a per-channel `bias` is added; the results for each channel are clamped to the range `[0, 1]`:

`A'= clamp(A * ScaleA + BiasA, 0, 1)`  
`R'= clamp(R * ScaleR + BiasR, 0, 1)`  
`G'= clamp(G * ScaleG + BiasG, 0, 1)`  
`B'= clamp(B * ScaleB + BiasB, 0, 1)`

Scale and bias values are input in floating point format but are then modified as follows:

 * `scale` parameters are clamped to the range `[-127.0, +127.0]`
 * `bias` parameters are clamped to the range `[-1.0, +1.0]`

The color transformation is set as a vector of 8 floats, consisting of the `R, G, B, A` scale factors followed by the `R, G, B, A` biases:

```c
/* Sr, Sg, Sb, Sa, Br, Bg, Bb, Ba */
VGfloat values[] = { 1.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0 };
vgSetfv(VG_COLOR_TRANSFORM_VALUES, 8, values);
vgSeti(VG_COLOR_TRANSFORM, VG_TRUE);
```

---

## Blending Equations [13.2]

Blending modes define alpha and color blending functions.  
Alpha blending function is formally defined as `α(αsrc, αdst)`.  
Color blending function is formally defined as `c(csrc, cdst, αsrc, αdst)`.  
Premultiplied alpha form is formally defined as `c'(αsrc * csrc, αdst * cdst, αsrc, αdst) = c'(c'src, c'dst, αsrc, αdst)` 
The `VGBlendMode` enumeration defines the possible blending modes:

| Blend Mode | Color blending function<br> `c'(c'src, c'dst, αsrc, αdst)` | Apha blending function `α(αsrc, αdst)` |
| ---------- | ---------------------------------------------------------- | -------------------------------------- |
| `VG_BLEND_SRC` | `c'src` | `αsrc` |
| `VG_BLEND_SRC_OVER` | `c'src + c'dst * (1 – αsrc)` | `αsrc + αdst * (1 – αsrc)` |
| `VG_BLEND_DST_OVER` | `c'src * (1 – αdst) + c'dst` | `αsrc * (1 – αdst) + αdst` |
| `VG_BLEND_SRC_IN` | `c'src * αdst` | `αsrc * αdst` |
| `VG_BLEND_DST_IN` | `c'dst * αsrc` | `αdst * αsrc` |
| `VG_BLEND_MULTIPLY` | `c'src * (1 - αdst) + c'dst * (1 - αsrc) + c'src * c'dst` | `αsrc + αdst * (1 – αsrc)` |
| `VG_BLEND_SCREEN` | `c'src + c'dst – c'src * c'dst` | `αsrc + αdst * (1 – αsrc)` |
| `VG_BLEND_DARKEN` | `min(c'src + c'dst * (1 – αsrc), c'dst + c'src * (1 – αdst))` | `αsrc + αdst * (1 – αsrc)` |
| `VG_BLEND_LIGHTEN` | `max(c'src + c'dst * (1 – αsrc), c'dst + c'src * (1 – αdst))` | `αsrc + αdst * (1 – αsrc)`
| `VG_BLEND_ADDITIVE` | `min(c'src + c'dst, 1)` | `min(αsrc + αdst, 1)` |
{:.rwd-table}

Example (set a "source over" blend mode):

```c
vgSeti(VG_BLEND_MODE, VG_BLEND_SRC_OVER);
```

---
