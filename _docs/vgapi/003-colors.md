---
layout: default-mj
title: "Colors"
date: 2017-01-01 08:00:00 +0100
chapter: 3
categories: [vgapi]
---

# Colors

## Colors [3.4]

Colors in OpenVG other than those stored in image pixels are represented as non-premultiplied sRGBA color values.
Color and alpha values lie in the range [0, 1] unless otherwise noted.

---

### Linear color space [3.4.2]

The linear lRGB color space is defined in terms of the standard CIE XYZ color space, following ITU Rec. 709 using a D65 white point:

$$ Rl = 3.240479 \ X \ – \ 1.537150 \ Y \ – \ 0.498535 \ Z $$

$$ Gl = -0.969256 \ X \ + \ 1.875992 \ Y \ + \ 0.041556 \ Z $$

$$ Bl = 0.055648 \ X \ – \ 0.204043 \ Y \ + \ 1.057311 \ Z $$

---

### sRGB color space [3.4.2]

The sRGB color space defines values Rs, Gs, Bs in terms of the linear lRGB primaries by applying a gamma $$ γ $$ mapping.

if $$ (x \le 0.00304) $$

$$ γ(x) = 12.92x $$

else  

$$ γ(x) = 1.0556 x^{1 / 2.4} \ – \ 0.0556 $$

if $$ (x \le 0.03928) $$  

$$ γ^{-1}(x) = x / 12.92 $$

else  

$$ γ^{-1}(x) = [(x + 0.0556) / 1.0556]^{2.4} $$  

lRGB to sRGB conversion $$ \qquad (1) $$

$$ Rs = γ(Rl) $$

$$ Gs = γ(Gl) $$

$$ Bs = γ(Bl) $$

sRGB to lRGB conversion $$ \qquad (2) $$

$$ Rl = γ^{-1}(Rs) $$

$$ Gl = γ^{-1}(Gs) $$

$$ Bl = γ^{-1}(Bs) $$

---

### Linear grayscale  

The linear grayscale (luminance) color space is related to the linear lRGB color space by the equations:

$$ lLUM = 0.2126 \ Rl \ + \ 0.7152 \ Gl \ + \ 0.0722 \ Bl \qquad (3) $$

$$ Rl = Gl = Bl = lLUM \qquad (4) $$

The perceptually-uniform grayscale color space is related to the linear grayscale (luminance) color space by the $$ γ $$ mapping:

$$ sLUM = γ(lLUM) \qquad (5) $$

$$ lLUM = γ^{-1}(sLUM) \qquad (6) $$

Conversion from perceptually-uniform grayscale to sRGB is performed by replication:

$$ Rs = Gs = Bs = LUMs \qquad (7) $$

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
{:.rwd-table}

---
