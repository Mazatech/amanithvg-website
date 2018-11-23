---
layout: default
title: "Extensions to the OpenVG API"
date: 2017-01-01 08:00:00 +0100
chapter: 4
categories: [desc]
headline: "AmanithVG OpenVG API proprietary extensions by Mazatech"
image: "separable_blend_modes.png"
keywords: "amanithvg API proprietary extensions conical gradient advanced blend modes separable color ramp interpolation cap style clip path mzt openvg"
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
The `VG_MZT_clip_path` extension extends the concept of clipping regions, giving the possibility to define them using `VGPath` objects. Clip paths could be pushed (see `vgClipPathPushMZT`) and popped (see `vgClipPathPopMZT`), in
a stack-like fashion: each drawing performed by `vgDrawPath` / `vgDrawImage` / `vgDrawGlyph` is clipped against the intersection of all pushed clip paths.

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
vgClipPathPushMZT(clipPath0);
vgDrawPath(drawPath, VG_FILL_PATH);

// push second clip path (intersection)
vgClipPathPushMZT(clipPath1);
vgDrawPath(drawPath, VG_FILL_PATH);

// in order to realize clip paths union, it is enough to create a single path
// and append other paths to it, using for example vgAppendPath or vgTransformPath
// functions, and taking care to set VG_NON_ZERO clip rule.
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
/* Push a new clip path. */
void vgClipPathPushMZT(VGPath path);

/* Pop out the last pushed clip path. */
void vgClipPathPopMZT(void);

/* Clear the whole clip paths queue. */
void vgClipPathClearMZT(void);
```

---
