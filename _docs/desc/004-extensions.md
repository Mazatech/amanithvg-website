---
layout: default
title: "Extensions to the OpenVG API"
date: 2018-11-01 08:00:00 +0100
chapter: 4
categories: [desc]
headline: "AmanithVG OpenVG API proprietary extensions by Mazatech"
image: "separable_blend_modes.png"
keywords: "amanithvg API proprietary extensions conical gradient advanced blend modes separable color ramp interpolation cap style clip path mask filters mzt openvg"
---

# MZT extensions

OpenVG provides a drawing model similar to those of existing 2D drawing APIs and formats (Adobe PostScript and PDF, Sun Microsystems Java2D, MacroMedia Flash, SVG). It is specifically intended to support all drawing features required by a SVG Tiny 1.2 renderer, and additionally to support functions that may be of use for implementing an SVG Basic renderer. In addition to the base feature set, we introduce a new set of interesting extensions; developers and designers can take advantage of these new features to develop their OpenVG applications. 

---

## Conical gradients [&nbsp;VG\_MZT\_conical\_gradient&nbsp;]

Conical gradients interpolate the color keys counter-clockwise around a point. A conical gradient is defined through the center point, the direction point and the number of repeats. Those points identify the line where the first color key lies on. The number of repeats tells how many times the circle will be split; all the keys take place every slice, following the classic spread mode rules.

This extension adds:

```c
VG_PAINT_CONICAL_GRADIENT_MZT = 0x1A90
```
to the `VGPaintParamType2Mzt` enum type.

```c
VG_PAINT_TYPE_CONICAL_GRADIENT_MZT = 0x1B90
```
to the `VGPaintTypeMzt` enum type.

| &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: |
| *Conical gradient, pad spread mode* | *Conical gradient, repeat spread mode* | *Conical gradient, reflect spread mode* |
{:.tbl_images .desc04_conical_gradient_spread}

```c
VGfloat congrad[5] = {
    center.x, center.y,
    target.x, target.y,
    repeats
};

vgSetParameteri(paint, VG_PAINT_TYPE, VG_PAINT_TYPE_CONICAL_GRADIENT_MZT);
vgSetParameterfv(paint, VG_PAINT_CONICAL_GRADIENT_MZT, 5, conGrad);
```

---

## Advanced blend modes [&nbsp;VG\_MZT\_advanced\_blend\_modes&nbsp;]

This extension completes the OpenVG 1.1 blend modes to support a full extended Porter-Duff rendering model (the same rendering model used by [SVG 1.2](http://www.w3.org/TR/2003/WD-SVG12-20030715/#compositing)).

This extension adds:

```c
VG_BLEND_CLEAR_MZT = 0x2090

VG_BLEND_DST_MZT = 0x2091

VG_BLEND_SRC_OUT_MZT = 0x2092

VG_BLEND_DST_OUT_MZT = 0x2093

VG_BLEND_SRC_ATOP_MZT = 0x2094

VG_BLEND_DST_ATOP_MZT = 0x2095

VG_BLEND_XOR_MZT = 0x2096

VG_BLEND_OVERLAY_MZT = 0x2097

VG_BLEND_COLOR_DODGE_MZT = 0x2098

VG_BLEND_COLOR_BURN_MZT = 0x2099

VG_BLEND_HARD_LIGHT_MZT = 0x209A

VG_BLEND_SOFT_LIGHT_MZT = 0x209B

VG_BLEND_DIFFERENCE_MZT = 0x209C

VG_BLEND_EXCLUSION_MZT = 0x209D
```
to the `VGBlendModeMzt` enum type.

Some of these new modes aren't available in AmanithVG GLE: *Overlay*, *Color Dodge*, *Color Burn*, *Hard Light*, *Soft Light*, *Difference*. In this case a *Src Over* fallback will be used.

| &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: | :---: | :---: | :---: |
| *Src* | *Dst* | *SrcOver* | *DstOver* | *SrcIn* | *DstIn* |
{:.tbl_images .desc04_blend01}

| &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: | :---: | :---: | :---: |
| *SrcOut* | *DstOut* | *SrcAtop* | *DstAtop* | *Clear* | *Xor* |
{:.tbl_images .desc04_blend02}

| &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: | :---: | :---: | :---: |
| *Screen* | *Multiply* | *Difference* | *Exclusion* | *Additive* | *Overlay* |
{:.tbl_images .desc04_blend03}

| &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: | :---: | :---: | :---: |
| *Darken* | *Lighten* | *ColorDodge* | *ColorBurn* | *HardLight* | *SoftLight* |
{:.tbl_images .desc04_blend04}

```c
vgSeti(VG_BLEND_MODE, VG_BLEND_EXCLUSION_MZT);
```

---

## Separable blend modes [&nbsp;VG\_MZT\_separable\_blend\_modes&nbsp;]

OpenVG 1.1 specifications provide a way to set a single blend mode, that will be used for both stroke and fill drawing. With this extension it is possible to independently specify a blend mode for the stroke and a blend mode for the fill.

This extension adds:

```c
VG_STROKE_BLEND_MODE_MZT = 0x1190

VG_FILL_BLEND_MODE_MZT = 0x1191
```
to the `VGParamType1Mzt` enum type.

| &nbsp; | 
| :---: |
| *SrcOver fill - Additive stroke* |
{:.tbl_images .desc04_separable_blend}

```c
vgSeti(VG_FILL_BLEND_MODE_MZT, VG_BLEND_SRC_OVER);
vgSeti(VG_STROKE_BLEND_MODE_MZT, VG_BLEND_ADDITIVE);
```

---

## Color ramp interpolation [&nbsp;VG\_MZT\_color\_ramp\_interpolation&nbsp;]

According to OpenVG 1.1 specifications, color and alpha values at offset values between the values given by stops are defined by means of linear interpolation between the values defined at the nearest stops above and below the given offset value. Linear interpolation suffers of the so called 'key highlights' issue; it is very noticeable when large surfaces are filled with a poor of keys gradient. This behaviour could be changed by defining a new color ramp interpolation schema. This extension introduces a smooth color interpolation, based on the Hermite interpolant coupled with Catmull-Rom tangents calculation. The result is a much smoother transition.

This extension adds:

```c
VG_PAINT_COLOR_RAMP_INTERPOLATION_TYPE_MZT = 0x1A91
```
to the `VGPaintParamType0Mzt` enum type.

the new `VGColorRampInterpolationTypeMzt` enum type, defined as:

```c
typedef enum {
    VG_COLOR_RAMP_INTERPOLATION_LINEAR_MZT = 0x1C90,
    VG_COLOR_RAMP_INTERPOLATION_SMOOTH_MZT = 0x1C91
} VGColorRampInterpolationTypeMzt;
```

| &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: |
| *Linear and smooth color interpolation* | *Radial gradient, linear color interpolation* | *Radial gradient, smooth color interpolation* |
{:.tbl_images .desc04_smooth_interpolation}

```c
vgSetParameteri(paint, VG_PAINT_COLOR_RAMP_INTERPOLATION_TYPE_MZT,
                       VG_COLOR_RAMP_INTERPOLATION_SMOOTH_MZT);
vgSetParameteri(paint, VG_PAINT_COLOR_RAMP_INTERPOLATION_TYPE_MZT,
                       VG_COLOR_RAMP_INTERPOLATION_LINEAR_MZT);
```

---

## Separable cap style [&nbsp;VG\_MZT\_separable\_cap\_style&nbsp;]

OpenVG 1.1 specifications provide a way to set a single cap style, that will be used for both start-cap and end-cap in a dashed stroke. With this extension it is possible to independently specify a different style for start-cap and end-cap.

This extension adds:

```c
VG_STROKE_START_CAP_STYLE_MZT = 0x1192

VG_STROKE_END_CAP_STYLE_MZT = 0x1193
```

to the official `VGParamType0Mzt` enum type.

| &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: |
| *Different start/end cap styles* | *Round start cap style, square end cap style* | *Square start cap style, round end cap style* |
{:.tbl_images .desc04_cap_style}

```c
vgSeti(VG_STROKE_START_CAP_STYLE_MZT, VG_CAP_ROUND);
vgSeti(VG_STROKE_END_CAP_STYLE_MZT, VG_CAP_SQUARE);
```

---

## Clip paths [&nbsp;VG\_MZT\_clip\_path&nbsp;] - SRE only

OpenVG 1.1 specifications provide a way to define a set of scissor rectangles: all drawing is clipped (i.e. restricted) to the surface sub-region defined by the union of such rectangles.
The `VG_MZT_clip_path` extension extends the concept of clipping regions, giving the possibility to define them using `VGPath` objects. Clip paths could be pushed (see `vgClipPathPushMZT`) and popped (see `vgClipPathPopMZT`), in a stack-like fashion: each drawing performed by `vgDrawPath` / `vgDrawImage` / `vgDrawGlyph` is clipped against the intersection of all pushed clip paths.

| &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: |
| *Clipping disabled* | *Clip against intersected paths* | *Clip against united paths* |
{:.tbl_images .desc04_clipping}

```c
// turn off clipping
vgSeti(VG_CLIPPING_MZT, VG_FALSE);
vgDrawPath(drawPath, VG_FILL_PATH);

// enable clipping
vgSeti(VG_CLIPPING_MZT, VG_TRUE);

// push first clip path
vgClipPathPushMZT(clipPath0, VG_TRUE);
vgDrawPath(drawPath, VG_FILL_PATH);

// push second clip path (intersection)
vgClipPathPushMZT(clipPath1, VG_TRUE);
vgDrawPath(drawPath, VG_FILL_PATH);

/*
    In order to realize clip paths union, it is enough to:

    1) set VG_NON_ZERO clip rule:
    vgSeti(VG_CLIP_RULE_MZT, VG_NON_ZERO);

    2) push the first clip path, establishing a new layer:
    vgClipPathPushMZT(path0, VG_TRUE);

    3) push all other paths on the current clip layer:
    vgClipPathPushMZT(path1, VG_FALSE);
    vgClipPathPushMZT(path2, VG_FALSE);
    vgClipPathPushMZT(path3, VG_FALSE);

    Be sure that all the pushed paths have the same orientation!
*/
```

This extension adds:

 * the new `VGParamType2Mzt` enum type, representing two new context parameters that can be get/set through `vgGet/vgSet` functions

```c
typedef enum {
    VG_CLIP_RULE_MZT = 0x1194,
    VG_CLIPPING_MZT = 0x1195
} VGParamType2Mzt;
```

 * a new matrix mode used to manipulate the transformation matrix associated to the clip paths

```c
typedef enum {
    VG_MATRIX_CLIP_USER_TO_SURFACE_MZT = 0x1405
} VGMatrixModeMzt;
```
 
 * three new functions used to push, pop and clear clip paths

```c
/*
    Push a new clip path.

    If 'advanceLayer' is VG_TRUE, a new clip layer is established, and the given
    path is "drawn" on it; the clip rule assigned to the new clip layer is the
    current value of VG_CLIP_RULE_MZT context parameter.

    If 'advanceLayer' is VG_FALSE, the given path is added to the current clip
    layer (if there is still no clip layer yet, a new clip layer is established
    with a clip rule equal to the current value of VG_CLIP_RULE_MZT context
    parameter).

    Possible errors:

    - VG_BAD_HANDLE_ERROR if path is not a valid path handle or if it is
      VG_INVALID_HANDLE, or is not shared with the current context

    - VG_OUT_OF_MEMORY_ERROR if we have already reached the maximum number of
      clip layers that can be established
*/
void vgClipPathPushMZT(VGPath path,
                       VGboolean advanceLayer);

/* Pop out the last pushed clip layer. */
void vgClipPathPopMZT(void);

/* Clear and remove all clip layers. */
void vgClipPathClearMZT(void);
```

---

## Extended alpha mask [&nbsp;VG\_MZT\_mask&nbsp;]

This extension adds a new function to modify the drawing surface mask values, in a very similar way to what the `vgMask` function does:

```c
void vgMaskMZT(VGHandle mask,
               VGMaskOperation operation,
               VGint x,
               VGint y,
               VGint width,
               VGint height);
```

If the given `mask` handle refers a `VGMaskLayer` or an image created with a single-channel format (`VG_sL_8`, `VG_lL_8`, `VG_A_8`, `VG_BW_1`, `VG_A_1`, `VG_A_4`), this function will behave as a standard `vgMask` call with the same given parameters. For all other image formats, the final mask value that will be applied to the OpenVG mask (according to the given operation) is computed as follow:

 - first a luminance value is computed from the color channel values RGB
 - then the computed luminance value is multiplied by the corresponding alpha value to produce the mask value

Such behavior is the one requested by the [SVG masking feature](https://www.w3.org/TR/SVG11/masking.html).

---

## Extended image filters [&nbsp;VG\_MZT\_filters&nbsp;]

This extension adds a new set of image filter functions:

 * `vgColorMatrixMZT`, equal to the standard `vgColorMatrix` filter, with the exception that images can overlap (e.g. they could be the same).

```c
void vgColorMatrixMZT(VGImage dst,
                      VGImage src,
                      const VGfloat* matrix);
```

 * `vgGaussianBlurMZT`, equal to the standard `vgGaussianBlur` filter, with the exception that images can overlap (e.g. they could be the same). If `axes` is NULL *and* `useFastApprox` is `VG_TRUE`, a fast Gaussian blur approximation (through subsequent application of separable box filters) is performed. If `axes` is non-NULL, a non-separable Gaussian blur is performed along the given `axes`: (`axes[0]`, `axes[1]`) defines the horizontal axis, (`axes[2]`, `axes[3]`) defines the vertical axis.

```c
void vgGaussianBlurMZT(VGImage dst,
                       VGImage src,
                       const VGfloat* axes,
                       VGfloat stdDeviationX,
                       VGfloat stdDeviationY,
                       VGTilingMode tilingMode,
                       VGboolean useFastApprox);
```

 * `vgLightingMZT`, it lights a source graphic using the alpha channel as a bump map.

```c
/*
    This filter lights a source graphic using the alpha channel as a bump map.
    The resulting image is an RGBA image based on the light color. The lighting
    calculation follows the standard specular component of the Phong lighting
    model. The resulting images depend on the light color, light position and
    surface geometry of the input bump map. The filter assumes that the viewer
    is at infinity in the z direction (i.e., the unit vector in the eye direction
    is (0, 0, 1) everywhere).

    'dstDiffuse' is the destination image for the diffuse component of the Phong
    lighting model:
    diffuse.r = Kd * <N, L> * light.color.r
    diffuse.g = Kd * <N, L> * light.color.g
    diffuse.b = Kd * <N, L> * light.color.b
    diffuse.a = 1

    The generated diffuse pixels are in the color space determined by the value of
    VG_FILTER_FORMAT_LINEAR (because alpha is always 1, the pixel can be thought
    of as both premultiplied and non-premultiplied), and then converted into the
    destination image space. This means that VG_FILTER_FORMAT_PREMULTIPLIED
    parameter is not actually used (it would not make sense to perform a useless
    intermediate conversion).

    'dstSpecular' is the destination image for the specular component of the Phong
    lighting model:
    specular.r = Ks * pow(<N, H>, light.specExp) * light.color.r
    specular.g = Ks * pow(<N, H>, light.specExp) * light.color.g
    specular.b = Ks * pow(<N, H>, light.specExp) * light.color.b
    specular.a = max(specular.r, specular.g, specular.b)

    The generated specular pixels are in the color space determined by the value of
    VG_FILTER_FORMAT_LINEAR with alpha-premultiplication enforced, and then converted
    into the destination image space. This means that VG_FILTER_FORMAT_PREMULTIPLIED
    parameter is not actually used (it would not make sense to perform a useless
    intermediate conversion).

    For the diffuse lighting, 'diffuseConstant' represents the Kd value in Phong
    lighting model, and must be non-negative. For the specular lighting:
    - 'specularConstant' represents the Ks value in Phong lighting model, and
       must be non-negative.
    - 'specularExponent' represents the exponent for specular term, larger is
       more "shiny", valid range is [1; 128]. Values outside the range are
       interpreted as the nearest endpoint of the range.

    The 'lightData' array contains the color and the geometric attributes of the
    light source:
    - [0] = red component of the light source
    - [1] = green component of the light source
    - [2] = blue component of the light source
    - [3+] = < geometric attributes of the light source >, variable length
             (see below)

    The color components of the light source are expressed in non-premultiplied
    sRGB, values outside the [0, 1] range are interpreted as the nearest endpoint
    of the range. According to the given 'lightType', the < geometric attributes
    of the light source > is a list of values as follows:

    - 2 entries for VG_LIGHT_TYPE_DISTANT_MZT light type
      [0] = azimuth angle, in degrees
      [1] = elevation angle, in degrees
    - 3 entries for VG_LIGHT_TYPE_POINT_MZT light type
      [0] = x location for the light source
      [1] = y location for the light source
      [2] = z location for the light source
    - 9 entries for VG_LIGHT_TYPE_SPOT_MZT light type
      [0] = x location for the light source
      [1] = y location for the light source
      [2] = z location for the light source
      [3] = x location of the point at which the light source is pointing
            (i.e. pointsAtX)
      [4] = y location of the point at which the light source is pointing
            (i.e. pointsAtY)
      [5] = z location of the point at which the light source is pointing
            (i.e. pointsAtZ)
      [6] = the exponent value controlling the focus for the light source
      [7] = the limiting cone angle which restricts the region where the light
            is projected, in degrees; valid range is [0; 90]
      [8] = smoothing threshold used to implement edge darkening at the
            boundary of the cone

    Possible errors:

    - VG_BAD_HANDLE_ERROR if 'src' is not a valid image handle, or is not
      shared with the current context

    - VG_BAD_HANDLE_ERROR if 'dstDiffuse' is different than VG_INVALID_HANDLE
      and is not a valid image handle or is not shared with the current context

    - VG_BAD_HANDLE_ERROR if 'dstSpecular' is different than VG_INVALID_HANDLE
      and is not a valid image handle or is not shared with the current context

    - VG_IMAGE_IN_USE_ERROR if either 'dstDiffuse', 'dstSpecular' or 'src' is
      currently a rendering target

    - VG_ILLEGAL_ARGUMENT_ERROR if 'src' and 'dstDiffuse' images overlap

    - VG_ILLEGAL_ARGUMENT_ERROR if 'src' and 'dstSpecular' images overlap

    - VG_ILLEGAL_ARGUMENT_ERROR if 'diffuseConstant' is less than zero

    - VG_ILLEGAL_ARGUMENT_ERROR if 'specularConstant' is less than zero

    - VG_ILLEGAL_ARGUMENT_ERROR if 'lightType' is not one of the values from
      the VGLightTypeMzt enumeration

    - VG_ILLEGAL_ARGUMENT_ERROR if 'lightData' is NULL or not properly aligned
*/
void vgLightingMZT(VGImage dstDiffuse,
                   VGImage dstSpecular,
                   VGImage src,
                   VGfloat surfaceScale,
                   // diffuse lighting
                   VGfloat diffuseConstant,
                   // specular lighting
                   VGfloat specularConstant,
                   VGfloat specularExponent,
                   // light information
                   VGLightTypeMzt lightType,
                   const VGfloat* lightData);
```

 * `vgMorphologyMZT`, it performs "fattening" or "thinning" of images.

```c
/*
    This filter performs "fattening" or "thinning" of images.
    The dilation (or erosion) kernel is a rectangle with a width of 2 * radiusX
    and a height of 2 * radiusY.

    In erosion ('erode' = VG_TRUE), the output pixel is the individual
    component-wise minimum of the corresponding R, G, B, A values in the
    source image's kernel rectangle.

    In dilation ('erode' = VG_FALSE), the output pixel is the individual
    component-wise maximum of the corresponding R, G, B, A values in the
    source image's kernel rectangle.

    Normally the canonical orthogonal axes (1, 0) - (0, 1) are used
    (i.e. NULL 'axes' argument) and in this case the filter implements a
    separable fast algorithm. It is possible to specify generic non-orthogonal
    'axes', in a such case the filter implements a slower non-separable algorithm.

    NB: 'src' and 'dst' images can overlap.

    Possible errors:

    - VG_BAD_HANDLE_ERROR if either 'dst' or 'src' is not a valid image
      handle, or is not shared with the current context

    - VG_IMAGE_IN_USE_ERROR if either 'dst' or 'src' is currently a rendering
      target

    - VG_ILLEGAL_ARGUMENT_ERROR if 'axes' is not NULL and not properly aligned

    - VG_ILLEGAL_ARGUMENT_ERROR if 'radiusX' or 'radiusY' is less than or equal
      to 0
*/
void vgMorphologyMZT(VGImage dst,
                     VGImage src,
                     VGboolean erode,
                     const VGfloat* axes,
                     VGint radiusX,
                     VGint radiusY);
```

 * `vgTurbulenceMZT`, it creates an image using the Perlin turbulence function.

```c
/*
    This filter creates an image using the Perlin turbulence function.
    It allows, for example, the synthesis of artificial textures like
    clouds or marble.
    
    The generated color and alpha values are in the color space determined
    by the value of VG_FILTER_FORMAT_LINEAR and VG_FILTER_FORMAT_PREMULTIPLIED.

    In order to generate (x, y) coordinates for noise generation, each pixel
    location (px, py) is shifted by 'bias' and multiplied by 'scale':

    noise.x = (pixel.x + biasX) * scaleX
    noise.y = (pixel.y + biasY) * scaleY

    An initial seed value is computed based on attribute 'seed'. Then the
    implementation computes the lattice points for R, then continues getting
    additional pseudo random numbers relative to the last generated pseudo
    random number and computes the lattice points for G, and so on for B and A.

    Possible errors:

    - VG_BAD_HANDLE_ERROR if 'image' is not a valid image handle, or is not
      shared with the current context

    - VG_IMAGE_IN_USE_ERROR if 'image' is currently a rendering target

    - VG_ILLEGAL_ARGUMENT_ERROR if 'baseFrequencyX' or 'baseFrequencyY' is
      less than 0

    - VG_ILLEGAL_ARGUMENT_ERROR if 'numOctaves' is less than or equal to 0
*/
void vgTurbulenceMZT(VGImage image,
                     VGfloat biasX,
                     VGfloat biasY,
                     VGfloat scaleX,
                     VGfloat scaleY,
                     VGfloat baseFrequencyX,
                     VGfloat baseFrequencyY,
                     VGint numOctaves,
                     VGint seed,
                     VGboolean stitchTiles,
                     VGboolean fractalNoise);
```

 * `vgDisplacementMapMZT`, it uses the pixels values from a map to spatially displace the source image.

```c
/*
    This filter uses the pixels values from 'map' to spatially displace the 'src'
    image; result is written to 'dst' image. This is the transformation to be
    performed:

    dst(x, y) = src(x + scaleX * (map(x, y, xChannelSelector) - 0.5),
                    y + scaleY * (map(x, y, yChannelSelector) - 0.5))

    where src(x, y) is the input image and dst(x, y) is the destination.
    map(x, y, xChannelSelector) and map(x, y, yChannelSelector) are the component
    values of the channel designated by the xChannelSelector and yChannelSelector.
    For example, to use the red component of 'map' to control displacement in x
    and the green component of 'map to control displacement in y, set
    'xChannelSelector' to VG_RED and 'yChannelSelector' to VG_GREEN.

    'map' pixels are read and converted to the space defined by the current
    values of VG_FILTER_FORMAT_PREMULTIPLIED and VG_FILTER_FORMAT_LINEAR
    parameters. Pixels read from 'src' image are then converted to the space
    of 'dst' image.

    Some mandatory preconditions:
    - 'src' and 'map' images must have the same dimensions
    - 'dst' and 'src' images cannot overlap
    - 'dst' and 'map' images cannot overlap

    NB: 'src' and 'map' images can overlap.

    Possible errors:

    - VG_BAD_HANDLE_ERROR if either 'dst', 'src' or 'map is not a valid image
      handle, or is not shared with the current context

    - VG_IMAGE_IN_USE_ERROR if either 'dst', 'src' or 'map' is currently a
      rendering target

    - VG_ILLEGAL_ARGUMENT_ERROR if 'dst' and 'src' overlap

    - VG_ILLEGAL_ARGUMENT_ERROR if 'dst' and 'map' overlap

    - VG_ILLEGAL_ARGUMENT_ERROR if 'src' and 'map' images do not have the same
      dimensions (i.e. different width or height)

    - VG_ILLEGAL_ARGUMENT_ERROR if 'tilingMode' is not one of the values from
      the VGTilingMode enumeration

    - VG_ILLEGAL_ARGUMENT_ERROR if either 'xChannelSelector' or 'yChannelSelector'
      is not one of the values from the VGImageChannel enumeration
*/
void vgDisplacementMapMZT(VGImage dst,
                          VGImage src,
                          VGImage map,
                          VGfloat scaleX,
                          VGfloat scaleY,
                          VGTilingMode tilingMode,
                          VGImageChannel xChannelSelector,
                          VGImageChannel yChannelSelector);
```

 * `vgCompositeMZT`, it composites two images together using commonly used blending modes.

```c
/*
    This filter composites two images together using commonly used blending
    modes: it performs a pixel-wise combination of two input images.
    Additionally, a component-wise arithmetic operation (with the result clamped
    between [0..1]) can be applied. If the arithmetic operation is chosen, each
    result pixel is computed using the following formula:

    dst(x, y) = k1 * in1(x, y) * in2(x, y) + k2 * in1(x, y) + k3 * in2(x, y) + k4

    'dst', 'in1', 'in2' images can overlap, but 'in1' and 'in2' images must have
    the same dimensions (mandatory precondition)

    Possible errors:

    - VG_BAD_HANDLE_ERROR if either 'dst', 'in1' or 'in2 is not a valid image
      handle, or is not shared with the current context

    - VG_IMAGE_IN_USE_ERROR if either 'dst', 'in1' or 'in2' is currently a
      rendering target

    - VG_ILLEGAL_ARGUMENT_ERROR if 'in1' and 'in2' images do not have the same
      dimensions (i.e. different width or height)

    - VG_ILLEGAL_ARGUMENT_ERROR if 'operation' is not one of the values from
      the VGCompositeOpMzt enumeration
*/
void vgCompositeMZT(VGImage dst,
                    VGImage in1,
                    VGImage in2,
                    VGCompositeOpMzt operation,
                    VGfloat k1,
                    VGfloat k2,
                    VGfloat k3,
                    VGfloat k4);
```

---
