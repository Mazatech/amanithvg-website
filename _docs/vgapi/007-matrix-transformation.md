---
layout: default
title: "Matrix transformation"
date: 2017-01-01 08:00:00 +0100
chapter: 7
categories: [vgapi]
headline: "Openvg API, matrix transformations"
image: "amanithvg-logo.png"
keywords: "openvg api matrix transformations translate scale rotate shear"
---

# Coordinate Systems and Trasformations

## Coordinate Systems [6.4]

Geometric coordinates are specified in the user coordinate system. The path-user-to-surface (`VG_MATRIX_PATH_USER_TO_SURFACE`)
and image-user-to-surface (`VG_MATRIX_IMAGE_USER_TO_SURFACE`) transformations map between the user coordinate system and pixel coordinates on the destination drawing surface.  
This pixel-based coordinate system is known as the surface coordinate system.  
The user coordinate system is oriented such that values along the x-axis increase from left to right and values along the y-axis increase from bottom to top.  
In the surface coordinate system, pixel `(0, 0)` is located at the lower-left corner of the drawing surface.

---

## Matrix Transformation [6.6]

### Select Matrix Mode

The current matrix to be manipulated is specified by setting the matrix mode. Separate matrices are maintained for transforming paths, images, and paint (gradients and patterns). The matrix modes are defined in the `VGMatrixMode` enumeration:

| Matrix mode | Type of matrix |
| ----------- | -------------- |
| `VG_MATRIX_PATH_USER_TO_SURFACE` | Affine |
| `VG_MATRIX_IMAGE_USER_TO_SURFACE` | Perspective |
| `VG_MATRIX_FILL_PAINT_TO_USER` | Affine |
| `VG_MATRIX_STROKE_PAINT_TO_USER` | Affine |
| `VG_MATRIX_GLYPH_USER_TO_SURFACE` | Affine |
{:.rwd-table .rwd-table-matrixMode}

To set the matrix mode, call `vgSeti` with a type of `VG_MATRIX_MODE` and a value of `VGMatrixMode`.  
For example, to set the matrix mode to allow manipulation of the path-user-to-surface transformation, call:

```c
vgSeti(VG_MATRIX_MODE, VG_MATRIX_PATH_USER_TO_SURFACE);
```

---

### Matrix Manipulation API

The matrix conventions used by OpenVG are similar to those of OpenGL. A point to be transformed is given by a homogeneous column vector **[x, y, 1]<sup>T</sup>**. Transformation of a point **p** by a matrix **M** is defined as the product **M &sdot; p**. Concatenation of transformations is performed using right-multiplication of matrices.

&nbsp;

| &nbsp; | 
| :---: |
| &nbsp; |
{:.tbl_images .matrix_transform} 

Matrix `m = { sx, shy, w0, shx, sy, w1, tx, ty, w2 }`

For affine transformations, `w0 = w1 = 0`, `w2 = 1`

```c
void vgLoadIdentity(void)
```
```c
void vgLoadMatrix(const VGfloat* m)
```
```c
void vgMultMatrix(const VGfloat* m)
```
```c
void vgGetMatrix(VGfloat* m)
```
```c
void vgTranslate(VGfloat tx, VGfloat ty)
```
```c
void vgScale(VGfloat sx, VGfloat sy)
```
```c
void vgShear(VGfloat shx, VGfloat shy)
```
```c
void vgRotate(VGfloat angle)
```

---
