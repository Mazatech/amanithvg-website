---
layout: default
title: "VGU utility library"
date: 2018-11-01 08:00:00 +0100
chapter: 15
categories: [vgapi]
headline: "Openvg API, VGU utility library"
image: "amanithvg-logo.png"
keywords: "openvg api vgu utility library"
---

# VGU Utility Library

For convenience, OpenVG provides an optional utility library known as VGU. Applications may choose whether to link to VGU at compile time; the library is not guaranteed to be present on the run-time platform. VGU is designed so it may be implemented in a portable manner using only the public functionality provided by the OpenVG library. VGU functions are defined in a vgu.h header file.

---

## Higher-level Geometric Primitives [17.1]

These functions allow applications to specify high-level geometric primitives to be appended to a path. Each primitive is reduced to a series of line segments, Bézier curves, and arcs. Input coordinates are mapped to input values for the `vgAppendPathData` command by subtracting the path’s `bias` and dividing by its `scale` value. Coordinates may overflow silently if the resulting values fall outside the range defined by the path datatype.

```c
vguErrorCode vguLine(VGPath path,
                     VGfloat x0, VGfloat y0,
                     VGfloat x1, VGfloat y1)
```
Appends a line segment to a `path`.

---
{:.hrlight}

```c
vguErrorCode vguPolygon(VGPath path,
                        const VGfloat* points,
                        VGint count,
                        VGboolean closed)
```
Appends a polyline (connected sequence of line segments) or polygon to a path.

---
{:.hrlight}

```c
vguErrorCode vguRect(VGPath path,
                     VGfloat x, VGfloat y,
                     VGfloat width, VGfloat height)
```
Appends an axis-aligned rectangle with its lower-left corner at `(x, y)` and a given `width` and `height` to a path.

---
{:.hrlight}

```c
vguErrorCode vguRoundRect(VGPath path,
                          VGfloat x, VGfloat y,
                          VGfloat width, VGfloat height,
                          VGfloat arcW, VGfloat arcH)
```
Appends an axis-aligned round-cornered rectangle with the lower-left corner of its rectangular bounding box at `(x, y)` and a given `width`, `height`, `arcWidth`, and `arcHeight` to a `path`.

---
{:.hrlight}

```c
vguErrorCode vguEllipse(VGPath path,
                        VGfloat cx, VGfloat cy, 
                        VGfloat width, VGfloat height)
```
Appends an axis-aligned ellipse to a `path`. The center of the ellipse is given by `(cx, cy)` and the dimensions of the axis-aligned rectangle enclosing the ellipse are given by `width` and `height`.

---
{:.hrlight}

```c
vguErrorCode vguArc(VGPath path,
                    VGfloat x, VGfloat y,
                    VGfloat width, VGfloat height,
                    VGfloat startAngle,
                    VGfloat angleExt,
                    VGUArcType arcType)
```
Appends an elliptical arc to a path, possibly along with one or two line segments, according to the `arcType` parameter.  
The `startAngle` and `angleExtent` parameters are given in degrees, proceeding counter-clockwise from the positive X axis.  
The `VGUArcType` enumeration defines three values to control the style of arcs:

| Arc type | Description |
| -------- | ----------- |
| `VGU_ARC_OPEN` | arc segment only |
| `VGU_ARC_CHORD` | arc, plus line between arc endpoints |
| `VGU_ARC_PIE` | arc, plus lines from each endpoint to the ellipse center |
{:.rwd-table .rwd-tableVguArc }

---

## Image Warping [17.2]

These functions compute 3x3 projective transform matrices. The first two compute the transformation from an arbitrary
quadrilateral onto the unit square, and vice versa. The third computes the transformation from an arbitrary quadrilateral
to an arbitrary quadrilateral.

```c
vguErrorCode vguComputeWarpQuadToSquare(VGfloat sx0, VGfloat sy0,
                                        VGfloat sx1, VGfloat sy1,
                                        VGfloat sx2, VGfloat sy2,
                                        VGfloat sx3, VGfloat sy3,
                                        VGfloat* matrix)
```

```c
vguErrorCode vguComputeWarpSquareToQuad(VGfloat dx0, VGfloat dy0,
                                        VGfloat dx1, VGfloat dy1,
                                        VGfloat dx2, VGfloat dy2,
                                        VGfloat dx3, VGfloat dy3,
                                        VGfloat* matrix)
```

```c
vguErrorCode vguComputeWarpQuadToQuad(VGfloat sx0, VGfloat sy0,
                                      VGfloat sx1, VGfloat sy1,
                                      VGfloat sx2, VGfloat sy2,
                                      VGfloat sx3, VGfloat sy3,
                                      VGfloat dx0, VGfloat dy0,
                                      VGfloat dx1, VGfloat dy1,
                                      VGfloat dx2, VGfloat dy2,
                                      VGfloat dx3, VGfloat dy3,
                                      VGfloat* matrix)
```

In all cases, if there is no projective mapping that satisfies the given constraints, or the mapping would be degenerate (i.e., non-invertible), `VGU_BAD_WARP_ERROR` is returned and `matrix` is unchanged.

---
