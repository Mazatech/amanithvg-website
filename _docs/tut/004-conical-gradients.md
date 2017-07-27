---
layout: default-mj
title: "Tut. 04 - Conical gradients"
date: 2017-01-01 08:00:00 +0100
chapter: 4
categories: [tut]
---

# Tutorial 04: Conical gradients

This tutorial will introduce a new type of gradient paint type, not covered by OpenVG specifications: conical gradients.
In order to extend the OpenVG API and add the support for conical gradients, new values for existing parameter types have been defined (see `vgext.h`), anyway the OpenVG pipeline behaviour remains the same. So lets start.

---

## Conical gradients in AmanithVG

Conical gradients define a scalar-valued gradient function based on a gradient center point `(cx, cy)`, a target point `(tx, ty)` and the number or `repeats`.
The scalar function has the following properties:

 - it is equal to `0` along the oriented direction `(cx, cy) - (fx, fy)` (i.e. from center *towards* target)

 - proceding counterclockwise, it is equal to the normalized angle (respect to the center-target direction) multiplied by the number of repeats

| ![gradFunc(x, y)]({{site.url}}/assets/images/tut04_congrad_func.png) | 
| :---: |
| *gradFunc(x, y)* | 

$$ gradFunc(x,y) = \left(\frac{\arctan\left(\frac{y \ - \ cy}{x \ - \ cx}\right)}{2\pi} \ - \ \frac{\arctan\left(\frac{ty \ - \ cy}{tx \ - \ cx}\right)}{2\pi}\right) \ \dot \ repeats $$

Such scalar value is mapped to colors by *color ramps*, exactly as it happens for [linear]({{site.url}}/docs/tut/002-linear-gradients.html) (have a look at it for more details) and [radial]({{site.url}}/docs/tut/003-radial-gradients.html) gradients (have a look at it for more details).

Geometric properties (center and target) defining a conical gradient are specified in the *paint coordinates system*; according to the OpenVG pipeline, such system is then transported to the path coordinates system throught a "paint-to-user" affine matrix (`VG_MATRIX_FILL_PAINT_TO_USER` / `VG_MATRIX_STROKE_PAINT_TO_USER`). Finally the filled/stroked path is moved to the drawing surface system by another affine matrix, the so called "path-user-to-surface" (`VG_MATRIX_PATH_USER_TO_SURFACE`).

| ![Gradient Matrices]({{site.url}}/assets/images/tut04_congrad_matrices.png) | 
| :---: |
| *Gradient Matrices* | 

To enable conical gradient paint, use `vgSetParameteri` to set the paint type to `VG_PAINT_TYPE_CONICAL_GRADIENT_MZT`.
The conical gradient parameters are set using `vgSetParameterfv` with a paramType argument of `VG_PAINT_CONICAL_GRADIENT_MZT`. The gradient values are supplied as a vector of `5` floats in the order `cx`, `cy`, `tx`, `ty`, `repeats`.

```c
// create a paint object
VGPaint conGradPaint = vgCreatePaint();
VGfloat conGradParams[5] = { cx, cy, tx, ty, repeats };
// set the paint to be a conical gradient
vgSetParameteri(conGradPaint, VG_PAINT_TYPE, VG_PAINT_TYPE_CONICAL_GRADIENT_MZT);
// set geometric parameters for the conical gradient (cx, cy), (tx, ty), repeats
vgSetParameterfv(conGradPaint, VG_PAINT_CONICAL_GRADIENT_MZT, 5, conGradParams);
```

Color ramps behaviour does not change for conical gradients, it's the same as specified for linear and radial gradients: `VG_PAINT_COLOR_RAMP_STOPS` parameter takes an array of floating-point values giving the offsets and colors of the stops, and all the three spread modes are supported too (`VG_COLOR_RAMP_SPREAD_PAD`, `VG_COLOR_RAMP_SPREAD_REPEAT`, `VG_COLOR_RAMP_SPREAD_REFLECT`).

| ![VG_COLOR_RAMP_SPREAD_PAD]({{site.url}}/assets/images/tut04_pad.png) | ![VG_COLOR_RAMP_SPREAD_REPEAT]({{site.url}}/assets/images/tut04_repeat.png) | ![VG_COLOR_RAMP_SPREAD_REFLECT]({{site.url}}/assets/images/tut04_reflect.png) |
| :---: | :---: | :---: |
| *Pad* | *Repeat* | *Reflect* |

In the pictures below you can see the visual meaning of conical gradient repeats.

| ![No repeats]({{site.url}}/assets/images/tut04_one_rep.png) | ![Two repeats]({{site.url}}/assets/images/tut04_two_rep.png) | ![Three repeats]({{site.url}}/assets/images/tut04_three_rep.png) |
| :---: | :---: | :---: |
| *No repeats* | *Two repeats* | *Three repeats* |

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
// concatenation of VG_MATRIX_FILL_PAINT_TO_USER and VG_MATRIX_PATH_USER_TO_SURFACE
PaintToSurface(paintPoint) = (paintPoint * userToSurfaceScale) + userToSurfaceTranslation
```

The inverse transformation maps a point from the drawing surface coordinates system to the paint system:

```c
// NB: in the general case we should have applied a matrix transformation given by the
// concatenation of inverse(VG_MATRIX_PATH_USER_TO_SURFACE) and inverse(VG_MATRIX_FILL_PAINT_TO_USER).
SurfaceToPaint(surfacePoint) = (surfacePoint - userToSurfaceTranslation) / userToSurfaceScale
```

Conical gradient repeats represent just a dimensionless number, so it remains the same regardless of the reference system.

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
Such geometric parameters are stored, in paint coordinates system, by three variables:

```c
// conical gradient parameters: center, target and repeats
float conGradCenter[2];
float conGradTarget[2];
float conGradRepeats;
```

 - every time we need to know their coordinates in surface space (e.g. when we want to draw them, within `tutorialDraw` function), we use the `PaintToSurface`mapping (see `gradientParamsGet` function)
 
 - every time we need to set a gradient control point in order to match the mouse/touch position (that is expressed in drawing surface space), we use the `SurfaceToPaint` mapping (see `gradientParamsSet` function)

The `VG_MZT_color_ramp_interpolation` extension, that we have seen in the previous tutorials, is supported by conical grandients too.

| ![VG_COLOR_RAMP_SPREAD_PAD]({{site.url}}/assets/images/tut04_pad.png) | ![VG_COLOR_RAMP_SPREAD_REPEAT]({{site.url}}/assets/images/tut04_smooth.png) |
| :---: | :---: |
| *Linear interpolation* | *Smooth interpolation* |

---

