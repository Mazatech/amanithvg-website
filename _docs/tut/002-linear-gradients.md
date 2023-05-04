---
layout: default
title: "Tut. 02 - Linear gradients"
date: 2018-11-01 08:00:00 +0100
chapter: 2
categories: [tut]
headline: "AmanithVG, how to create and use linear gradient paint type"
image: "tut02_smooth_dark.png"
keywords: "amanithvg tutorial 02 linear gradient pad repeat reflect openvg"
---

# Tutorial 02: Linear gradients

Using OpenVG API it is possible to draw vector paths, specifying a paint for the fill (i.e.: the bounded inside region defined by its contours) and a paint for the stroke (i.e. the "widening" of path edges using a straight-line pen).
The are a total of four possible paint types: plain colors, linear gradients, radial gradients and bitmap patterns. AmanithVG extends such paint types with an additional one: conical gradients (see [`VG_MZT_conical_gradient` extension]({{site.url}}/docs/desc/004-extensions.html)).
This tutorial will introduce linear gradients.

---

## Linear gradients in OpenVG

Linear gradients define a scalar-valued gradient function based on two points `(x0, y0)` and `(x1, y1)` with the following properties:

 - it is equal to `0` at `(x0, y0)`

 - it is equal to `1` at `(x1, y1)`

 - it increases linearly along the line from `(x0, y0)` to `(x1, y1)`

 - it is constant along lines perpendicular to the line from `(x0, y0)` to `(x1, y1)`

Such scalar value is mapped to colors by *color ramps*. The application defines the non-premultiplied sRGBA color and alpha value associated with each of a number of values, called *stops*.
A stop is defined by an offset between `0` and `1` and a color value. Color and alpha values at offset values between the stops are defined by means of linear interpolation between color values defined at the nearest stops above and below the given offset value.

The two geometric points `(x0, y0)` and `(x1, y1)` that defines a linear gradient are specified in the *paint coordinates system*; according to the OpenVG pipeline, such system is then transported to the path coordinates system throught a "paint-to-user" affine matrix (`VG_MATRIX_FILL_PAINT_TO_USER` / `VG_MATRIX_STROKE_PAINT_TO_USER`). Finally the filled/stroked path is moved to the drawing surface system by another affine matrix, the so called "path-user-to-surface" (`VG_MATRIX_PATH_USER_TO_SURFACE`).

| &nbsp; | 
| :---: |
| *Gradient Matrices* |
{:.tbl_images .tut02_grad_matrices}

To enable linear gradient paint, use `vgSetParameteri` to set the paint type to `VG_PAINT_TYPE_LINEAR_GRADIENT`.
The linear gradient parameters are set using `vgSetParameterfv` with a `paramType` argument of `VG_PAINT_LINEAR_GRADIENT`. The gradient values are supplied as a vector of 4 floats in the order `x0`, `y0`, `x1`, `y1`.

```c
// create a paint object
VGPaint linGradPaint = vgCreatePaint();
VGfloat linGradParams[4] = { x0, y0, x1, y1 };
// set the paint to be a linear gradient
vgSetParameteri(linGradPaint, VG_PAINT_TYPE, VG_PAINT_TYPE_LINEAR_GRADIENT);
// set geometric parameters for the linear gradient (x0, y0) and (x1, y1)
vgSetParameterfv(linGradPaint, VG_PAINT_LINEAR_GRADIENT, 4, linGradParams);
```

---

## Color ramps

Color ramp parameters are set using `vgSetParameter`.

The `VG_PAINT_COLOR_RAMP_SPREAD_MODE` parameter controls the spread mode, that is how the given set of stops are repeated or extended in order to define interpolated color values for arbitrary input values outside the `[0; 1]` range. The are three modes:

 - `VG_COLOR_RAMP_SPREAD_PAD`: extend stops

 - `VG_COLOR_RAMP_SPREAD_REPEAT`: repeat stops

 - `VG_COLOR_RAMP_SPREAD_REFLECT`: repeat stops in reflected order

| &nbsp; | &nbsp; | &nbsp; |
| :---: | :---: | :---: |
| *Pad* | *Repeat* | *Reflect* |
{:.tbl_images .tut02_color_ramp}

The `VG_PAINT_COLOR_RAMP_STOPS` parameter takes an array of floating-point values giving the offsets and colors of the stops, in order.
Each stop is defined by a floating-point offset value and four floating-point values containing the sRGBA color and alpha value associated with each stop, in the form of a non-premultiplied `(R, G, B, α)`.

```c
VGfloat colStops[25] = {
    // ofs   R     G      B      α
    // ----------------------------
    0.00f, 0.40f, 0.00f, 0.60f, 1.00f,
    0.25f, 0.90f, 0.50f, 0.10f, 1.00f,
    0.50f, 0.80f, 0.80f, 0.00f, 1.00f,
    0.75f, 0.00f, 0.30f, 0.50f, 1.00f,
    1.00f, 0.40f, 0.00f, 0.60f, 1.00f
};
// set color stops
vgSetParameterfv(linGradPaint, VG_PAINT_COLOR_RAMP_STOPS, 25, colStops);
// set gradient spread mode
vgSetParameteri(linGradPaint, VG_PAINT_COLOR_RAMP_SPREAD_MODE,
                              VG_COLOR_RAMP_SPREAD_REPEAT);
```

---

## The tutorial code

In order to simplify both code and understanding, we will set the `VG_MATRIX_FILL_PAINT_TO_USER` matrix to the identity: so the paint coordinates system will coincide with the path-user one.
An additional simplification is given by setting the `VG_MATRIX_PATH_USER_TO_SURFACE` matrix as a unifrom scale plus a translation only (see '`tutorialDraw`' function):

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
// concatenation of inverse(VG_MATRIX_PATH_USER_TO_SURFACE) and 
// inverse(VG_MATRIX_FILL_PAINT_TO_USER) matrices.
SurfaceToPaint(surfacePoint) = (surfacePoint - userToSurfaceTranslation) / userToSurfaceScale
```

The tutorial draws a circle path at the center of drawing surface, filled with a linear gradient.
The path is created in object space with center at `(0, 0)` and a radius of `1` (see `genPaths` function):

```c
// create the circle that will be filled by the linear gradient paint
filledCircle = vgCreatePath(VG_PATH_FORMAT_STANDARD, VG_PATH_DATATYPE_F,
                            1.0f, 0.0f, 0, 0, VG_PATH_CAPABILITY_ALL);
vguEllipse(filledCircle, 0.0f, 0.0f, 2.0f, 2.0f);
```

By having defined the path in this tricky way, it's really easy to scale and center it in order to fit a rectangular region (i.e. the drawing surface):

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

Gradient control points are highlighted by two white spots that can be moved by using mouse/touch.
Their position, in paint coordinates system, is kept by two variables, `linGradStart` and `linGradEnd`:

```c
// linear gradient parameters (x0, y0) and (x1, y1)
float linGradStart[2];
float linGradEnd[2];
```

 - every time we need to know their coordinates in surface space (e.g. when we want to draw them, within `tutorialDraw` function), we use the `PaintToSurface` mapping (see `gradientParamsGet` function)
 
 - every time we need to set a gradient control point in order to match the mouse/touch position (that is expressed in drawing surface space), we use the `SurfaceToPaint` mapping (see `gradientParamsSet` function)

---

## Working with OpenVG extensions

This tutorial also shows the standard way of working with extensions. OpenVG contains a mechanism for applications to access information about the runtime platform: the `vgGetString` function returns information about the OpenVG implementation, including extension information. In this tutorial we are interested to verify the runtime support of `VG_MZT_color_ramp_interpolation` extension (see [AmanithVG extensions](http://www.amanithvg.com/docs/desc/004-extensions.html)):

```c
void extensionsCheck(void) {
    // get the space-separated list of supported OpenVG extensions
    const char* extensions = (const char*)vgGetString(VG_EXTENSIONS);
    // check for the support of VG_MZT_color_ramp_interpolation extension
    smoothRampSupported = extensionFind("VG_MZT_color_ramp_interpolation", extensions);
}
```

The `VG_MZT_color_ramp_interpolation` extension introduces a smooth color ramp interpolation schema, based on the Hermite interpolant coupled with Catmull-Rom tangents calculation. The result is a much smoother transition in colored gradients:

| &nbsp; | &nbsp; |
| :---: | :---: |
| *Linear interpolation* | *Smooth interpolation* |
{:.tbl_images .tut02_color_ramp_interpolation}

---
