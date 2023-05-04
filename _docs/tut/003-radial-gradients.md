---
layout: default
title: "Tut. 03 - Radial gradients"
date: 2018-11-01 08:00:00 +0100
chapter: 3
categories: [tut]
headline: "AmanithVG, how to create and use radial gradient paint type"
image: "tut03_repeat_dark.png"
keywords: "amanithvg tutorial 03 radial gradient pad repeat reflect openvg"
---

# Tutorial 03: Radial gradients

In the [previous tutorial]({{site.url}}/docs/tut/002-linear-gradients.html) we have introduced linear gradients, the simplest gradient paint type provided by the OpenVG API.
In this tutorial we will have a look at a much more complex gradient type: radial gradients.

---

## Radial gradients in OpenVG

Radial gradients define a scalar-valued gradient function based on a gradient circle defined by a center point `(cx, cy)`, a radius `r`, and a focal point `(fx, fy)` that is forced to lie within the circle.
The scalar function has the following properties:

 - it is equal to `0` at the focal point `(fx, fy)`

 - it is equal to `1` along the circumference of the gradient circle

 - elsewhere, it is equal to the distance between `(x, y)` and `(fx, fy)` divided by the length of the line segment starting at `(fx, fy)`, passing through `(x, y)`, and ending on the circumference of the gradient circle

(side note: if the gradient radius is less than or equal to `0`, the function is given the value `1` everywhere)

| &nbsp; |
| :---: |
| *gradFunc(x, y) = D / (D + d)* |
{:.tbl_images .tut03_grad_func} 

Such scalar value is mapped to colors by *color ramps*, exactly as it happens for [linear gradients]({{site.url}}/docs/tut/002-linear-gradients.html) (have a look at it for more details).

Geometric properties (center, focus and radius) defining a radial gradient are specified in the *paint coordinates system*; according to the OpenVG pipeline, such system is then transported to the path coordinates system throught a "paint-to-user" affine matrix (`VG_MATRIX_FILL_PAINT_TO_USER` / `VG_MATRIX_STROKE_PAINT_TO_USER`). Finally the filled/stroked path is moved to the drawing surface system by another affine matrix, the so called "path-user-to-surface" (`VG_MATRIX_PATH_USER_TO_SURFACE`).

| &nbsp; |
| :---: |
| *Gradient Matrices* | 
{:.tbl_images .tut03_grad_matrices} 

To enable radial gradient paint, use `vgSetParameteri` to set the paint type to `VG_PAINT_TYPE_RADIAL_GRADIENT`.
The radial gradient parameters are set using `vgSetParameterfv` with a `paramType` argument of `VG_PAINT_RADIAL_GRADIENT`. The gradient values are supplied as a vector of 5 floats in the order `cx`, `cy`, `fx`, `fy`, `r`.

```c
// create a paint object
VGPaint radGradPaint = vgCreatePaint();
VGfloat radGradParams[5] = { cx, cy, fx, fy, r };
// set the paint to be a radial gradient
vgSetParameteri(radGradPaint, VG_PAINT_TYPE, VG_PAINT_TYPE_RADIAL_GRADIENT);
// set geometric parameters for the radial gradient (cx, cy), (fx, fy), r
vgSetParameterfv(radGradPaint, VG_PAINT_RADIAL_GRADIENT, 5, radGradParams);
```

Color ramps behaviour does not change for radial gradients, it's the same as specified for linear gradients: `VG_PAINT_COLOR_RAMP_STOPS` parameter takes an array of floating-point values giving the offsets and colors of the stops, and all the three spread modes are supported too (`VG_COLOR_RAMP_SPREAD_PAD`, `VG_COLOR_RAMP_SPREAD_REPEAT`, `VG_COLOR_RAMP_SPREAD_REFLECT`).

| &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: |
| *Pad* | *Repeat* | *Reflect* |
{:.tbl_images .tut03_color_ramp}

---

## The tutorial code

In order to simplify both code and understanding, we will set the `VG_MATRIX_FILL_PAINT_TO_USER` matrix to the identity: so the paint coordinates system will coincide with the path-user one.
An additional simplification is given by setting the `VG_MATRIX_PATH_USER_TO_SURFACE` matrix as a unifrom scale plus a translation only (see `tutorialDraw` function):

```c
vgSeti(VG_MATRIX_MODE, VG_MATRIX_PATH_USER_TO_SURFACE);
vgLoadIdentity();
vgTranslate(userToSurfaceTranslation[X], userToSurfaceTranslation[Y]);
vgScale(userToSurfaceScale, userToSurfaceScale);
```

So we can implement a simple map from the paint coordinates system to the drawing surface system as follow:

```c
// Map a point from the paint coordinates system to the drawing surface system.
// NB: in the general case we should have applied a matrix transformation given by the
// concatenation of VG_MATRIX_FILL_PAINT_TO_USER and VG_MATRIX_PATH_USER_TO_SURFACE.
PaintToSurface(paintPoint) = (paintPoint * userToSurfaceScale) + userToSurfaceTranslation
```

The inverse transformation maps a point from the drawing surface coordinates system to the paint system:

```c
// NB: in the general case we should have applied a matrix transformation given by the
// concatenation of inverse(VG_MATRIX_PATH_USER_TO_SURFACE) and inverse(VG_MATRIX_FILL_PAINT_TO_USER) matrices.
SurfaceToPaint(surfacePoint) = (surfacePoint - userToSurfaceTranslation) / userToSurfaceScale
```

Instead, a different approach must be taken for the gradient radius: the radius represents the value of a distance from the gradient center point, so it is *translation agnostic*.
This implies that given a radius in the paint coordinates system, its value in the drawing surface system is simply calculated by multiplying (or dividing, if going from surface to paint system) it by the scale factor only:

```c
PaintToSurface(paintRadius) = paintRadius * userToSurfaceScale
SurfaceToPaint(surfaceRadius) = surfaceRadius / userToSurfaceScale
```

The tutorial draws a circle path at the center of drawing surface, filled with a radial gradient.
The path is created in object space with center at `(0, 0)` and a radius of `1` (see `genPaths` function):

```c
// create the circle that will be filled by the radial gradient paint
filledCircle = vgCreatePath(VG_PATH_FORMAT_STANDARD, VG_PATH_DATATYPE_F, 
                            1.0f, 0.0f, 0, 0, VG_PATH_CAPABILITY_ALL);
vguEllipse(filledCircle, 0.0f, 0.0f, 2.0f, 2.0f);
```

By having defined the path in this tricky way, it's really easy to scale and center it in order to fit the drawing surface:

```c
// calculate "path user to surface" transformation
void userToSurfaceCalc(const int surfaceWidth,
                       const int surfaceHeight) {
    // find the minimum dimension between surface width and height, then halve it
    int halfDim = (surfaceWidth < surfaceHeight) ? (surfaceWidth / 2)
                                                 : (surfaceHeight / 2);
    // calculate scale factor in order to cover 90% of it
    userToSurfaceScale = halfDim * 0.9f;
    // translate to the surface center
    userToSurfaceTranslation[X] = surfaceWidth / 2;
    userToSurfaceTranslation[Y] = surfaceHeight / 2;
}
```

Gradient control points (i.e. center and focus) are highlighted by two white spots that can be moved by using mouse/touch.
Even the gradient radius (highlighted by a white circle) can be enlarged or shrinked by dragging it.
Such geometric parameters are stored, in paint coordinates system, by three variables:

```c
// radial gradient parameters: center, focus and radius
float radGradCenter[2];
float radGradFocus[2];
float radGradRadius;
```

 - every time we need to know their coordinates in surface space (e.g. when we want to draw them, within `tutorialDraw` function), we use the `PaintToSurface` mapping (see `gradientParamsGet` function)

 - every time we need to set a gradient control point (or the radius) in order to match the mouse/touch position (that is expressed in drawing surface space), we use the `SurfaceToPaint` mapping (see `gradientParamsSet` function)

The `VG_MZT_color_ramp_interpolation` extension, that we have seen in the previous tutorial, is supported by radial grandients too.

| &nbsp; | &nbsp; |
| :---: | :---: |
| *Linear interpolation* | *Smooth interpolation* |
{:.tbl_images .tut03_color_ramp_interpolation}


---
