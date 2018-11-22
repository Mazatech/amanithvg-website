---
layout: default
title: "Tut. 06 - Stroking"
date: 2017-01-01 08:00:00 +0100
chapter: 6
categories: [tut]
headline: "AmanithVG, how to stroke a path with solid or dashing stroke, and different join and cap styles"
image: "tut06_solid_stroke_dark.png"
keywords: "amanithvg tutorial 06 path stroke stroking join bevel round miter cap butt square solid dashing openvg"
---

# Tutorial 06: Stroking

Using OpenVG API it is possible to draw vector paths by filling and/or stroking them.
The concept of path filling is really simple:

 - a non-self-intersecting closed path divides the plane into two regions, a bounded *inside* region and an unbounded *outside* region
 
 - a path that self-intersects, or that has multiple overlapping subpaths, requires additional information in order to define the *inside* region; such information is called *fill rule*

Path filling coincides with the filling of the *inside* region.
Path stroking, instead, is a more complex topic, so lets introduce it.

---

## Path stroking in OpenVG

Stroking a path consists of "widening" the edges of the path using a straight-line pen held perpendicularly to the path. In addition:

 - at the start and end vertices of the path, an additional end-cap style is applied
 
 - at interior vertices of the path, a line join style is applied
 
 - at a cusp of a Bézier segment, the pen is rotated smoothly between the incoming and outgoing tangents

Conceptually, stroking of a path is performed in two steps. First, the stroke parameters are applied in the user coordinate system to form a new shape representing the end result of dashing, widening the path, and applying the end cap and line join styles. Second, a path is created that defines the outline of this stroked shape. This path is transformed using the path-user-to-surface transformation (`VG_MATRIX_PATH_USER_TO_SURFACE` matrix). Finally, the resulting path is filled with paint in exactly the same manner as when filling a user-defined path using the non-zero fill rule.

---

## Stroke parameters

Stroking a path involves the following parameters, set on the current OpenVG context:

 - line width in user coordinate system units
 
 - end cap style: one of `VG_CAP_BUTT`, `VG_CAP_ROUND`, or `VG_CAP_SQUARE`
 
 - line join style: one of `VG_JOIN_MITER`, `VG_JOIN_ROUND`, or `VG_JOIN_BEVEL`
 
 - miter limit, if using `VG_JOIN_MITER` join style
 
 - dash pattern, an array of dash on/off lengths in user units
 
 - dash phase, an initial offset into the dash pattern
 
These parameters are set using the variants of the `vgSet` function: the values most recently set prior to calling `vgDrawPath` are applied to generate the stroke. Here's an example:

```c
// a simple dash pattern "on", "off", "on", "off"
float dashPattern[4] = { 5.0f, 55.0f, 35.0f, 35.0f };
// stroke parameters
vgSetf(VG_STROKE_LINE_WIDTH, 12.0f);
vgSeti(VG_STROKE_CAP_STYLE, VG_CAP_ROUND);
vgSeti(VG_STROKE_JOIN_STYLE, VG_JOIN_MITER);
vgSetf(VG_STROKE_MITER_LIMIT, 4.0f);
vgSetfv(VG_STROKE_DASH_PATTERN, 4, dashPattern);
vgSetf(VG_STROKE_DASH_PHASE, 0.0f);
vgSeti(VG_STROKE_DASH_PHASE_RESET, VG_TRUE);
vgDrawPath(clover, VG_STROKE_PATH);
```

| &nbsp; |
| :---: |
| *Setting stroke parameters* | 
{:.tbl_images .tut06_stroke_parameters} 

---

## End cap styles

 - the `VG_CAP_BUTT` end cap style terminates each segment with a line perpendicular to the tangent at each endpoint

 - the `VG_CAP_ROUND` end cap style appends a semicircle with a diameter equal to the line width centered around each endpoint

 - the `VG_CAP_SQUARE` end cap style appends a rectangle with two sides of length equal to the line width perpendicular to the tangent, and two sides of length equal to half the line width parallel to the tangent, at each endpoint. The outgoing tangent is used at the left endpoint and the incoming tangent is used at the right endpoint

| !&nbsp; |
| :---: |
| *End cap styles* |
{:.tbl_images .tut06_cap_styles} 

---

## Line join styles

 - the `VG_JOIN_BEVEL` join style appends a triangle with two vertices at the outer endpoints of the two "fattened" lines and a third vertex at the intersection point of the two original lines

 - the `VG_JOIN_ROUND` join style appends a wedge-shaped portion of a circle, centered at the intersection point of the two original lines, having a radius equal to half the line width

 - the `VG_JOIN_MITER` join style appends a trapezoid with one vertex at the intersection point of the two original lines, two adjacent vertices at the outer endpoints of the two "fattened" lines and a fourth vertex at the extrapolated intersection point of the outer perimeters of the two "fattened" lines

| &nbsp; | 
| :---: |
| *Join styles* |
{:.tbl_images .tut06_join_styles} 

The ratio of miter length to line width may be computed directly from the angle `θ` between the two line segments being joined as `1 / sin(θ/2)`. A number of angles with their
corresponding miter limits for a line width of `1` are shown in the following table:

| Angle (degrees) | Miter limit |
| :-------------: | :---------: |
| 10.00 | 11.47 |
| 11.47 | 10.00 |
| 23.00 | 5.00 |
| 28.95 | 4.00 |
| 30.00 | 3.86 |
| 38.94 | 3.00 |
| 45.00 | 2.61 |
| 60.00 | 2.00 |
| 90.00 | 1.41 |
| 120.00 | 1.15 |
| 150.00 | 1.03 |
| 180.00 | 1.00 |
{:.rwd-table}

---

## Dashing

The dash pattern consists of a sequence of lengths of alternating "on" and "off" dash segments. The first value of the dash array defines the length, in user coordinates, of the first "on" dash segment. The second value defines the length of the following "off" segment. Each subsequent pair of values defines one "on" and one "off" segment. The dash phase defines the starting point in the dash pattern that is associated with the start of the first segment of the path (a negative dash phase is equivalent to the positive phase obtained by adding a suitable multiple of the dash pattern length).

For example, if the dash pattern is `[ 10 20 30 40 ]` and the dash phase is `35`, the path will be stroked with an "on" segment of length `25` (skipping the first "on" segment of length `10`, the following "off" segment of length `20`, and the first `5` units of the next "on" segment), followed by an "off" segment of length `40`. The pattern will then repeat from the beginning, with an "on" segment of length `10`, an "off" segment of length `20`, an "on" segment of length `30`, etc.

| &nbsp; | 
| :---: |
| *Dashing* |
{:.tbl_images .tut06_dashing} 

Conceptually, dashing is performed by breaking the path into a set of subpaths according to the dash pattern. Each subpath is then drawn independently using the end cap, line join style, and miter limit that were set for the path as a whole.

---

## The tutorial code

The tutorial draws a clover-like path at the center of drawing surface, stroked with a plain color.
The path, consisting in four cubic Bézier segments, is defined in user-space within a square region having a side of `512` units: center `(0, 0)`, lower-left corner `(-256, -256)`, upper-right corner `(256, 256)`.

By having defined the path in this way, it's really easy to scale and center it in order to fit the drawing surface:

```c
// find the minimum dimension between surface width and height, then halve it
int halfDim = (surfaceWidth < surfaceHeight) ? (surfaceWidth / 2) : (surfaceHeight / 2);
// calculate scale factor in order to cover 90% of it
userToSurfaceScale = (halfDim / 256.0f) * 0.9f;
// translate to the surface center
userToSurfaceTranslation[X] = surfaceWidth / 2;
userToSurfaceTranslation[Y] = surfaceHeight / 2;

// "user to surface" transformation (a uniform scale plus a translation),
// upload the matrix to the OpenVG backend
vgSeti(VG_MATRIX_MODE, VG_MATRIX_PATH_USER_TO_SURFACE);
vgLoadIdentity();
vgTranslate(userToSurfaceTranslation[X], userToSurfaceTranslation[Y]);
vgScale(userToSurfaceScale, userToSurfaceScale);
```

So we can implement a simple map from the path coordinates system to the drawing surface system as follow:

```c
// Map a point from the path coordinates system to the drawing surface system.
PathToSurface(pathPoint) = (pathPoint * userToSurfaceScale) + userToSurfaceTranslation
```

The inverse transformation maps a point from the drawing surface coordinates system to the path system:

```c
// Map a point from the drawing surface system to the path reference system.
SurfaceToPath(surfacePoint) = (surfacePoint - userToSurfaceTranslation) / userToSurfaceScale
```

The path is drawn using the current stroke parameters; their values are stored to global variables and uploaded to the OpenVG backend when needed:

```c
// available dash patterns
const float dashPatterns[4][4] = {
    { 30.0f, 30.0f,  5.0f, 45.0f },
    { 15.0f, 40.0f, 15.0f, 40.0f },
    { 20.0f, 25.0f, 20.0f, 45.0f },
    {  5.0f, 45.0f, 35.0f, 25.0f }
};

// an index pointing to one of the four available 
// dash patterns (i.e. valid values: 0, 1, 2, 3)
int dashPattern;
// dash phase
float dashPhase;
// join style
VGJoinStyle joinStyle;
// cap style
VGCapStyle capStyle;
```

The user can play with such stroke parameters and, at the same time, move path control points. About the control points:

 - every time we need to know their coordinates in surface space (e.g. when we want to draw them, within `tutorialDraw` function), we use the `PathToSurface` mapping (see `controlPointGet` function)
 
 - every time we need to set a path control point in order to match the mouse/touch position (that is expressed in drawing surface space), we use the `SurfaceToPath` mapping (see `controlPointSet` function)

| &nbsp; | 
| :---: |
| *Solid stroke in the tutorial app* |
{:.tbl_images .tut06_solid_stroke} 

---

## Stroke extensions

AmanithVG implements an extension to the OpenVG API relative to stroke parameters: `VG_MZT_separable_cap_style` (see [extensions]({{site.url}}/docs/desc/004-extensions.html) chapter for additional details).
OpenVG specifications provide a way to set a single cap style, that will be used for both start-cap and end-cap in a dashed stroke. Through this extension, instead, it is possible to independently specify a different style for start-cap and end-cap.

The tutorial application will check (at runtime) the support of `VG_MZT_separable_cap_style` extension and, if found, will enable the user to modify both start-cap and end-cap styles.

```c
// global variables used to store cap styles
VGCapStyle startCapStyle;
VGCapStyle endCapStyle;

void extensionsCheck(void) {
    // get the list of supported OpenVG extensions
    const char* extensions = (const char*)vgGetString(VG_EXTENSIONS);
    // check for the support of VG_MZT_separable_cap_style extension
    separableCapsSupported = extensionFind("VG_MZT_separable_cap_style", extensions);
}

// set cap style(s)
if (separableCapsSupported) {
    vgSeti(VG_STROKE_START_CAP_STYLE_MZT, startCapStyle);
    vgSeti(VG_STROKE_END_CAP_STYLE_MZT, endCapStyle);
}
else {
    vgSeti(VG_STROKE_CAP_STYLE, startCapStyle);
}
```

| &nbsp; | 
| :---: |
| *start cap butt, end cap round* |
{:.tbl_images .tut06_stroke_ext} 

---