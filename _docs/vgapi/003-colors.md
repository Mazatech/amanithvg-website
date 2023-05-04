---
layout: default
title: "Colors"
date: 2018-11-01 08:00:00 +0100
chapter: 3
categories: [vgapi]
headline: "Openvg API, color spaces and relative conversions"
image: "amanithvg-logo.png"
keywords: "openvg api color spaces conversions linear sRGB luminance"
---

# Colors

## Colors [3.4]

Colors in OpenVG other than those stored in image pixels are represented as non-premultiplied sRGBA color values.
Color and alpha values lie in the range `[0, 1]` unless otherwise noted.

---

### Linear color space [3.4.2]

The linear lRGB color space is defined in terms of the standard CIE XYZ color space, following ITU Rec. 709 using a D65 white point:

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_xyz_to_linear}

---

### sRGB color space [3.4.2]

The sRGB color space defines values Rs, Gs, Bs in terms of the linear lRGB primaries by applying a gamma **γ** mapping.

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_srgb_eq}

lRGB to sRGB conversion:

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_lrgb_to_srgb}

sRGB to lRGB conversion:

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_srgb_to_lrgb}

---

### Linear grayscale  

The linear grayscale (luminance) color space is related to the linear lRGB color space by the equations:

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_lrgb_to_lum}

The perceptually-uniform grayscale color space is related to the linear grayscale (luminance) color space by the gamma **γ** mapping:

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_lum_conversions}

Conversion from perceptually-uniform grayscale to sRGB is performed by replication:

&nbsp;

| &nbsp; | 
| :-----: |
| &nbsp; |
{:.tbl_images .color_lum_to_srgb}

---

### Color space conversions  

The following table summarizes the steps needed to convert one color space into another.
The source format is in the left column, and the destination format is in the top row. The
numbers indicate the equations from this section that are to be applied, in left-to-right order:

| Src/Dst | lRGB | sRGB | lLUM | sLUM |
| ------- | ---- | ---- | ---- | ---- |
| lRGB | - | 1 | 3 | 3, 5 |
| sRGB | 2 | - | 2, 3 | 2, 3, 5 |
| lLUM | 4 | 4, 1 | - | 5 |
| sLUM | 7, 2 | 7 | 6 | - |
{:.rwd-table .rwd-table-colorConversions}

---
