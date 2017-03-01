---
layout: default
title: "Drawing context"
date: 2017-01-01 08:00:00 +0100
chapter: 4
categories: [vgapi]
---

# Drawing Context

## Api States [4]

OpenVG functions that perform drawing, or that modify or query drawing state make use of an implicit drawing context. A context is created, attached to a drawing surface, and bound to a running application thread outside the scope of the OpenVG API. OpenVG API calls are executed with respect to the context currently bound to the thread in which they are executed.  
When an image, paint, path, font, or mask handle is defined, it is permanently attached to the context that is current at that time.
The context is responsible for maintaining the OpenVG API state, as shown in the table:

| State                                 | Description                                           |
| ------------------------------------- | ----------------------------------------------------- |
| Drawing Surface                       | Surface for drawing                                   |
| Matrix Mode                           | Transformation to be manipulated                      |
| Path user-to-surface Transformation   | Affine transformation for filled and stroked geometry |
| Image user-to-surface Transformation  | Affine or projective transformation for images        |
| Paint-to-user Transformations         | Affine transformations for paint applied to geometry  | 
| Glyph user-to-surface Transformation  | Affine transformation for glyphs                      |
| Glyph origin                          | (X,Y) origin of a glyph to be drawn                   |
| Fill Rule                             | Rule for filling paths                                |
| Quality Settings                      | Image and rendering quality, pixel layout             |
| Color Transformation                  | Color transformation coefficients and enable/disable  |
| Blend Mode                            | Pixel blend function                                  |
| Image Mode                            | Image/paint combination function                      |
| Scissoring                            | Current scissoring rectangles and enable/disable      |
| Stroke                                | Stroke parameters                                     |
| Pixel and Screen layout               | Pixel layout information                              |
| Tile fill color                       | Color for FILL tiling mode                            |
| Clear color                           | Color for fast clear                                  |
| Filter Parameters                     | Image filtering parameters                            |
| Paint                                 | Paint definitions                                     |
| Mask                                  | Coverage mask and enable/disable                      |
| Error                                 | Oldest unreported error code                          |
{:.rwd-table}

---

## Forcing Drawing to Complete API [4.3]

```c
void vgFlush(void)
```

Ensure that all outstanding requests on the current context will complete in finite time.
This function may return prior to the actual completion of all requests.

---
{:.hrlight}

```c
void vgFinish(void)
```

Force all outstanding requests on the current context to complete, returning only when the last request has completed.

---
