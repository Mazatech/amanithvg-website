---
layout: default
title: "Implementation details"
date: 2017-01-01 08:00:00 +0100
chapter: 2
categories: [desc]
headline: "AmanithVG implementation details, requirements and limitations"
image: "amanithvg-logo.png"
keywords: "amanithvg sre gle engines 2d vector graphics implementation details openvg"
---

# Implementation details

AmanithVG SRE and AmanithVG GLE implement the whole OpenVG 1.0.1 and OpenVG 1.1 features, without limitations; while for pure software rendering solutions this is quite common, for solutions that relay on top of OpenGL ES 1.x+ this is an hard task. 
AmanithVG GLE is one of the few OpenVG engines that accomplishes this task using 2 texture units and one auxiliary buffer (depth or stencil) only, using few software fallbacks where specific GL extensions aren't available and just to realize a couple of features. 

| OpenVG feature | AmanithVG SRE | AmanithVG GLE |
| :--- | :---: | :---: |
| Scissoring | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Clearing | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Alpha Masking | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__ |
| Matrices & transformations | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Paths | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__ |
| Fill & stroke (solid, dashed) | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Fill rule | <span class="green_text">__CPU__</span> | <span class="red_text">__CPU__</span> |
| Paints (color, gradient, pattern) | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Color ramp spread modes | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Pattern tiling modes | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Images | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Image modes (normal, multiply, stencil) | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Image quality | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Filters | <span class="green_text">__CPU__</span> | <span class="red_text">__CPU__</span> |
| Rendering quality | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span>\* |
| Blend modes | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span>\*\* |
| Color transform | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span>\*\*\* |
| Font & text | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span> |
| Color spaces | <span class="green_text">__CPU__</span> | <span class="green_text">__GPU__</span>\*\*\*\* |
| Vgu | <span class="green_text">__CPU__</span> | <span class="red_text">__CPU__</span> |
{:.rwd-table}

| :--- |
| *\* antialiasing relays on GL multisampling.* |
| *\*\* darken & lighten without GL\_EXT\_blend\_minmax are realized on CPU.* |
| *\*\*\* color transform on images in multiply mode, when bias != 0, are realized on CPU.* |
| *\*\*\*\* except unpremultiplied drawing surfaces format.* |
{:.rwd-table}

---

## Requirements and limitations

AmanithVG SRE is a single thread library and it's not thread safe.
AmanithVG uses 32 bit single precision floating point arithmetic, as defined by IEEE 754-1985 standard, to perform some internal computations. All input data that describe geometries such as (but not limited to) path coordinates, paint parameters and matrices, will be converted from relative to absolute forms (if applicable), normalized (if applicable) and internally stored as a 32 bit floating point number. After these steps, it could result that different input values will become the same value under the 32 bit floating point precision.

AmanithVG uses fixed point arithmetic to perform the rasterization; in this case internal rendering surface coordinates are represented using 16bit. Fixed point format can be configured (at compile time) as:

 * 10.6 (10 bits for the integer part, 6 bits for the fractional part): supported maximum rendering surface dimension is 1024 pixels
 
 * 11.5 (11 bits for the integer part, 5 bits for the fractional part): supported maximum rendering surface dimension is 2048 pixels
 
 * 12.4 (12 bits for the integer part, 4 bits for the fractional part): supported maximum rendering surface dimension is 4096 pixels

AmanithVG supports rendering surfaces with dimensions less than or equal to 4096 pixels only.

---

## GLE specific requirements and limitations

AmanithVG GLE is a single thread library, it's not thread safe, it doesn't support multiple rendering context at the same time; so it can be bound a single OpenGL / OpenGL ES rendering context / rendering surface and, on it, a single OpenVG context at once.

AmanithVG GLE uses integer arithmetic to perform robust polygon triangulation; in this case internalnormalized object space coordinates (relative to the axes-aligned bounding box of the polygon) are represented using 16 bits or 21 bits (this option is customizable at compile time).

The drawing surface content persistence is the same of the underlying GL drawing surface; this means thatafter a “swap buffers” operation on GL Graphic System with non-persistent buffers, the buffers content couldn't be consistent. On such cases, it's highly recommendable to perform a vgClear of the whole drawing surface (disabling the scissoring), as the first OpenVG operation after the `vgPostSwapBuffersMZT` call.

Edge antialiasing (`VGRenderingQuality`): AmanithVG GLE has no control over edge antialiasing, so VGRenderingQuality values (`VG_RENDERING_QUALITY_NONANTIALIASED`, `VG_RENDERING_QUALITY_FASTER`, `VG_RENDERING_QUALITY_BETTER`) have no effect.

---

### OpenGL and OpenGL ES requirements and limitations

AmanithVG GLE uses standard OpenGL ES functions available since OpenGL ES 1.0 version, and standard OpenGL functions available since OpenGL 1.2 taking advantage of the following extensions if supported by the GL Graphic System:

 * `GL_EXT_texture_rectangle` or `GL_ARB_texture_rectangle`
 
 * `GL_ARB_texture_mirrored_repeat` or `GL_OES_texture_mirrored_repeat`
 
 * `GL_ARB_texture_border_clamp`
 
 * `GL_EXT_blend_minmax` or `GL_ARB_blend_minmax`
 
 * `GL_EXT_texture_env_dot3` or `GL_ARB_texture_env_dot3`
 
 * `GL_ARB_vertex_buffer_object`

To run, AmanithVG GLE requires at least:

 * a GL Graphic System equipped with 1 texture unit,

 * a GL rendering surface created with color buffer attribute and,

 * at least, depth or stencil buffer attributes

Furthermore, to ensure an OpenVG rendering output as described, AmanithVG GLE requires also:

 * a GL Graphic System equipped with 2 or more texture units

 * a maximum OpenGL / OpenGL ES texture size greater or equal than the maximum OpenGL / OpenGL ES rendering surface dimension (width or 
 height). If lesser, alpha mask will be downsampled to the maximum texture size.
 
 * a maximum OpenGL / OpenGL ES texture size greater or equal than the maximum dimension of the axis-aligned bounding box (in screen space and clipped against the rendering surface boundaries) of the drawn OpenVG primitive (image or path, including the stroke). If lesser, pixels contained inside the region defined by the primitive bounding box will be downsampled to the maximum texture size, in order to perform drawings that require multipass techniques.

The following limitations will affect some OpenVG features:

 * if `VG_MZT_advanced_blend_modes` extension is available, the following blend modes are silently replaced by `VG_BLEND_SRC_OVER` blend mode:
 
   * `VG_BLEND_OVERLAY`,
 
   * `VG_BLEND_COLOR_DODGE`,
 
   * `VG_BLEND_COLOR_BURN`,
 
   * `VG_BLEND_HARD_LIGHT`,
 
   * `VG_BLEND_SOFT_LIGHT`,
 
   * `VG_BLEND_DIFFERENCE`.
 
   * if `GL_EXT_texture_env_dot3` and `GL_ARB_texture_env_dot3` extensions are not available, `VG_DRAW_IMAGE_STENCIL` image mode is silently replaced by `VG_DRAW_IMAGE_MULTIPLY`.

 * if the GL Graphic System is equipped with 1 texture unit only:
 
   * drawing a path with paint type different than `VG_PAINT_TYPE_COLOR` and alpha masking enabled, the alpha mask is silently disabled
 
   * drawing an image with alpha masking enabled, the alpha mask is silently disabled
 
   * drawing an image in `VG_DRAW_IMAGE_MULTIPLY` image mode and paint type different than `VG_PAINT_TYPE_COLOR`, the image mode is silently replaced by `VG_DRAW_IMAGE_NORMAL`.

 * if the GL Graphic System supports OpenGL ES 1.0+ CL only, AmanithVG GLE will use 16bit normalized object space coordinates to perform the polygon triangulation, and it will use 16.16 fixed point format to represent object space coordinates and matrices.

 * AmanithVG GLE uses `GL_RGBA` texture format for the rendering of non-opaque (considering also the effect of color transform) images, patterns and gradients. The pixels buffers (32bit R8G8B8A8) are uploaded onto textures using the following function:

```c
glTexImage2D(target, 0, GL_RGBA, w, h, 0, GL_RGBA, GL_UNSIGNED_BYTE, pixels)
```

If the GL Graphic System internally stores textures using less than 32bit per-pixel, there will be a visual quality loss (e.g. gradients band effect).

AmanithVG GLE may provide software rendering fallbacks to realize OpenVG features that cannot be accelerated by the GL Graphic System.

---
