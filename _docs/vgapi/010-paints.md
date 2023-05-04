---
layout: default
title: "Paints"
date: 2018-11-01 08:00:00 +0100
chapter: 10
categories: [vgapi]
headline: "Openvg API, paints definition"
image: "amanithvg-logo.png"
keywords: "openvg api paint paints"
---

# Paints

Paint defines a color and an alpha value for each pixel being drawn. Color paint defines a constant color for all pixels; gradient paint defines a linear or radial pattern of smoothly varying colors; and pattern paint defines a possibly repeating rectangular pattern of colors based on a source image. Paint is defined in its own coordinate system, which is transformed into user coordinates by means of the fill-paint-to-user (`VG_MATRIX_FILL_PAINT_TO_USER`) and stroke-paint-to-user (`VG_MATRIX_STROKE_PAINT_TO_USER`) transformations, depending on whether the current geometry is being filled or stroked.

---

## Paint Definition [9.1]

`VGPaint` represents an opaque handle to a paint object.
Changes to a `VGPaint` object (e.g. using `vgSetParameter`) attached to a context will immediately affect drawing calls on that context.

```c
typedef VGHandle VGPaint;
```

---

### Create & Destroy Paint [9.1.1]

```c
VGPaint vgCreatePaint(void)
```

Create a new paint object that is initialized to a set of default values and returns a `VGPaint` handle to it.

---
{:.hrlight}

```c
void vgDestroyPaint(VGPaint paint)
```

Deallocate a paint, releasing any resources associated with it; the paint handle is no longer valid in any
of the contexts that shared it. If the paint object is currently active in a drawing context, the context
continues to access it until it is replaced or the context is destroyed.

---

### Set the Current Paint [9.1.2]

```c
void vgSetPaint(VGPaint paint, VGbitfield paintModes)
```

Set a paint on the current context. The `paintModes` argument is a bitwise OR of values from the `VGPaintMode` enumeration, determining whether the paint object is to be used for filling (`VG_FILL_PATH`), stroking (`VG_STROKE_PATH`), or both (`VG_FILL_PATH | VG_STROKE_PATH`). The specified paint replaces the previously set paint object, if any, for the given paint mode or modes.

---
{:.hrlight}

```c
VGPaint vgGetPaint(VGPaintMode paintMode)
```
Return the paint object currently set for the given `paintMode`.

---

### Paint Object Parameter [9.1.3]

Values from the `VGPaintParamType` enumeration may be used as the `paramType` argument to `vgSetParameter` and `vgGetParameter` to set and query various features of a paint object. For each parameter, default values are shown in <span class="ovg_default">YELLOW</span>.

| Parameter name | Parameter type | Possible values / Notes |
| -------------- | -------------- | ----------------------- |
| `VG_PAINT_TYPE` | `VGPaintType` | <span class="ovg_default">`VG_PAINT_TYPE_COLOR`</span><br>`VG_PAINT_TYPE_LINEAR_GRADIENT`<br>`VG_PAINT_TYPE_RADIAL_GRADIENT`<br>`VG_PAINT_TYPE_PATTERN` |
| `VG_PAINT_COLOR` | `VGfloat[4]` | Format is `{ red, green, blue, alpha }` sRGBA<br>Values outside the `[0, 1]` range are interpreted as the nearest endpoint of the range<br><span class="ovg_default">`{ 0.0f, 0.0f, 0.0f, 1.0f }`</span> |
| `VG_PAINT_COLOR_RAMP_SPREAD_MODE` | `VGColorRampSpreadMode` | <span class="ovg_default">`VG_COLOR_RAMP_SPREAD_PAD`</span><br>`VG_COLOR_RAMP_SPREAD_REPEAT`<br>`VG_COLOR_RAMP_SPREAD_REFLECT` |
| `VG_PAINT_COLOR_RAMP_PREMULTIPLIED` | `VGboolean` | <span class="ovg_default">`VG_TRUE`</span><br>`VG_FALSE` (disabled) |
| `VG_PAINT_COLOR_RAMP_STOPS` | `VGfloat*` | Format is `{ offset0, red0, green0, blue0, alpha0, ... }`<br>Color components are expressed in sRGBA space (`[0, 1]` range)<br>Stops with offsets <0 or >1 are ignored<br><span class="ovg_default">`NULL`</span> |
| `VG_PAINT_LINEAR_GRADIENT` | `VGfloat[4]` | Format is `{ startx, starty, endx, endy }`<br><span class="ovg_default">`{ 0.0f, 0.0f, 1.0f, 0.0f }`</span> |
| `VG_PAINT_RADIAL_GRADIENT` | `VGfloat[5]` | Format is `{ centerx, centery, focusx, focusy, radius }`<br>If `(focusx, focusy)` lies outside the circumference of the circle, the intersection of the line from the center to the focal point with the circumference of the circle is used as the focal point in place of the specified point<br><span class="ovg_default">`{ 0.0f, 0.0f, 0.0f, 0.0f, 1.0f }`</span> |
| `VG_PAINT_PATTERN_TILING_MODE` | `VGTilingMode` | <span class="ovg_default">`VG_TILE_FILL`</span><br>`VG_TILE_PAD`<br>`VG_TILE_REPEAT`<br>`VG_TILE_REFLECT` |
{:.rwd-table .rwd-paintParameters}

---

## Color Paint [9.2]

Color paint uses a fixed color and alpha for all pixels. An alpha value of 1 produces a fully opaque color. Colors are specified in non-premultiplied sRGBA format.

Example (set an opaque red):  

```c
VGfloat col[4] = { 1.0, 0.0, 0.0, 1.0f };
vgSetParameterfv(VGPaint paint, VG_PAINT_COLOR, 4, col);
```

A shorthand function that allows the `VG_PAINT_COLOR` parameter of a given paint object to be set using a 32-bit non-premultiplied `sRGBA_8888` representation. The rgba parameter is a `VGuint` with 8 bits of red starting at the most significant bit, followed by 8 bits each of green, blue, and alpha.
Each color or alpha channel value is conceptually divided by 255.0f to obtain a value between 0 and 1.

```c
void vgSetColor(VGPaint paint, VGuint rgba)
```

Example (set an opaque red):

```c
vgSetColor(paint, 0xFF0000FF);
```

---
{:.hrlight}

Get the current setting of the `VG_PAINT_COLOR` parameter on a given paint object; returned value is a 32-bit non-premultiplied `sRGBA_8888` value.
Each color channel or alpha value is clamped to the range [0, 1] , multiplied by 255, and rounded to obtain an 8-bit integer; the resulting values are packed into a 32-bit value in the same format as for `vgSetColor`.

```c
VGuint vgGetColor(VGPaint paint)
```

---

## Gradient Paint [9.3]

Gradients are patterns used for filling or stroking. They are defined mathematically in two parts; a scalar-valued gradient function defined at every point in the two-dimensional plane (in paint coordinates), followed by a color ramp mapping.

---

### Linear Gradients [9.3.1]

Linear gradients define a scalar-valued gradient function based on two points (x0, y0) and (x1, y1) (in the paint coordinate system) with the following properties:

 * it is equal to 0 at (x0, y0)
 * it is equal to 1 at (x1, y1)
 * it increases linearly along the line from (x0, y0) to (x1, y1)
 * it is constant along lines perpendicular to the line from (x0, y0) to (x1, y1)

To enable linear gradient paint, use `vgSetParameteri` to set the paint type to `VG_PAINT_TYPE_LINEAR_GRADIENT`.
The linear gradient parameters are set using `vgSetParameterfv` with a `paramType` argument of `VG_PAINT_LINEAR_GRADIENT`.
The gradient values are supplied as a vector of 4 floats in the order `{ x0, y0, x1, y1 }`.

Example (set a linear gradient, going from opaque red to a transparent green):

```c
VGfloat stops[10] = {
    0.0f, 1.0f, 0.0f, 0.0f, 1.0f,  // opaque red
    1.0f, 0.0f, 1.0f, 0.0f, 0.0f   // transparent green
};

VGfloat linearGradient[4] = {
    0.0f, 0.0f,    // start point
    256.0f, 256.0f // end point
};

vgSetParameteri(paint, VG_PAINT_TYPE, VG_PAINT_TYPE_LINEAR_GRADIENT);
vgSetParameterfv(paint, VG_PAINT_COLOR_RAMP_STOPS, 10, stops);
vgSetParameterfv(paint, VG_PAINT_LINEAR_GRADIENT, 4, linearGradient);
vgSetParameteri(paint, VG_PAINT_COLOR_RAMP_SPREAD_MODE, VG_COLOR_RAMP_SPREAD_PAD);
```

---

### Radial Gradients [9.3.2]

Radial gradients define a scalar-valued gradient function based on a gradient circle defined by a center point `(cx, cy)`, a radius `r`, and a focal point `(fx, fy)` that is forced to lie within the circle. All parameters are given in the paint coordinate system. The radial gradient function is:

 * equal to 0 at the focal point
 * equal to 1 along the circumference of the gradient circle
 * elsewhere, equal to the distance between `(x, y)` and `(fx, fy)` divided by the length of the line segment starting at `(fx, fy)`, passing through `(x, y)`, and ending on the circumference of the gradient circle

To enable radial gradient paint, use `vgSetParameteri` to set the paint type to `VG_PAINT_TYPE_RADIAL_GRADIENT`. The radial gradient parameters are set using
`vgSetParameterfv` with a `paramType` argument of `VG_PAINT_RADIAL_GRADIENT`. The gradient values are supplied as a vector of 5 floats in the order `{ cx, cy, fx, fy, r }`.

Example (set a radial gradient, going from opaque red to a transparent blue):

```c
VGfloat stops[10] = {
    0.0f, 1.0f, 0.0f, 0.0f, 1.0f,  // opaque red
    1.0f, 0.0f, 0.0f, 1.0f, 0.0f   // transparent blue
};

VGfloat radialGradient[5] = {
    256.0f, 256.0f,  // center point
    200.0f, 200.0f,  // focus point
    100.0f           // radius
};

vgSetParameteri(paint, VG_PAINT_TYPE, VG_PAINT_TYPE_RADIAL_GRADIENT);
vgSetParameterfv(paint, VG_PAINT_COLOR_RAMP_STOPS, 10, stops);
vgSetParameterfv(paint, VG_PAINT_RADIAL_GRADIENT, 4, radialGradient);
vgSetParameteri(paint, VG_PAINT_COLOR_RAMP_SPREAD_MODE, VG_COLOR_RAMP_SPREAD_REFLECT);
```

---

## Pattern Paint [9.4]

Pattern paint defines a rectangular pattern of colors based on the pixel values of an image. Each pixel `(x, y)` of the pattern image defines a point of color at the pixel center `(x + 0.5, y + 0.5)`. The pattern tiling mode is used to define values for pixel centers in the pattern space that lie outside of the bounds of the pattern.
The `vgPaintPattern` function replaces any previous pattern image defined on the given paint object for the given set of paint modes with a new pattern image.
If the current paint object has its `VG_PAINT_TYPE` parameter set to `VG_PAINT_TYPE_PATTERN`, but no pattern image is set, the paint object behaves as if `VG_PAINT_TYPE` were set to `VG_PAINT_TYPE_COLOR`.

```c
void vgPaintPattern(VGPaint paint, VGImage pattern)
```

Example (set a pattern paint):

```c
#define PATTERN_WIDTH 64
#define PATTERN_HEIGHT 64

VGImage img = vgCreateImage(VG_sRGBA_8888, 
                            PATTERN_WIDTH, 
                            PATTERN_HEIGHT, 
                            VG_IMAGE_QUALITY_FASTER);

VGuint* pixels = (VGuint*)malloc(PATTERN_WIDTH * PATTERN_HEIGHT * sizeof(VGuint));

for (VGuint i = 0; i < PATTERN_WIDTH * PATTERN_HEIGHT; ++i) {
  pixels[i] = ((VGuint)rand() << 16) | rand();
}

vgImageSubData(img, (const void *)pixels,
                    PATTERN_WIDTH * sizeof(VGuint),
                    VG_sRGBA_8888,
                    0, 0, 
                    PATTERN_WIDTH, PATTERN_HEIGHT);
vgSetParameteri(paint, VG_PAINT_TYPE, VG_PAINT_TYPE_PATTERN);
vgSetParameteri(paint, VG_PAINT_PATTERN_TILING_MODE, VG_TILE_PAD);
vgPaintPattern(paint, img);
free(pixels);
```

---
