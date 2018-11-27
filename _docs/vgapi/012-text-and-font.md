---
layout: default
title: "Text and font"
date: 2018-11-01 08:00:00 +0100
chapter: 12
categories: [vgapi]
headline: "Openvg API, glyph, font and text definition"
image: "amanithvg-logo.png"
keywords: "openvg api text font glyph fonts glyphs"
---

# Text and Font Operations

OpenVG provides a mechanism to allow applications to define a `VGFont` object as a collection of glyphs, where each glyph can be represented as either a `VGPath` representing either an original unhinted outline that can be scaled and rendered, or a scaled and hinted outline; or a `VGImage` representing a scaled, optionally hinted, and rendered image of a glyph.

---

## Font Definition [11.4]

`VGFont` objects are created by an application, and can contain either a full set of glyphs or a subset of glyphs of an original font. `VGFont` objects do not contain any metric or layout information; instead, applications are responsible for all text layout operations using the information provided by the original fonts. A `VGFont` is an opaque handle to a font object.

```c
typedef VGHandle VGFont;
```

---

### Glyph Mapping [11.4.1]

Glyphs in a `VGFont` are identified by a glyph index, which is an arbitrary number assigned to a glyph when it is created. The semantics of the mapping are application-dependent. Possible mappings include:

 * __Unicode character codes__  
 When a `VGFont` is created as a subset that supports only simple language scripts (e.g., Latin), the character code values may be used as glyph indices. This eliminates the need for an additional mapping table: a text string may be passed directly as an argument (as an array of glyph indices) to OpenVG API call for text rendering.

 * __Native font glyph indices__  
 OpenVG applications may re-use native glyph indices from an original TrueType or OpenType font when `VGFont` object is created. This simplifies text composition and layout decisions by re-using OpenType/TrueType layout and character-to-glyph mapping tables.

 * __Application-defined (custom) glyph indices__   
 OpenVG applications may assign arbitrary numbers as glyph indices. This may be beneficial for special purpose fonts that have a limited number of glyphs (e.g., SVG fonts).

---

### Manage VGFont Object [11.4.2]

```c
VGFont vgCreateFont(VGint glyphCapacityHint)
```

Create a new font object and return a `VGFont` handle to it.  
The `glyphCapacityHint` argument provides a hint as to the capacity of a `VGFont`.  
A value of `0` indicates that the value is unknown.

---
{:.hrlight}

```c
void vgDestroyFont(VGFont font)
```

Destroy the given `VGFont` object: underlying objects that were used to define glyphs in the font won't be destroyed.  
It is the responsibility of an application to destroy all `VGPath` or `VGImage` objects that were used in a `VGFont`, if they are no longer in use.

---

### Font Object Parameter [11.4.3]

Values from the `VGFontParamType` enumeration can be used as the `paramType` argument to `vgGetParameter` to query font features. All of the parameters defined by `VGFontParamType` are read-only.

| Parameter name | Parameter type | Possible values / Notes |
| -------------- | -------------- | ----------------------- |
| `VG_FONT_NUM_GLYPHS` | `VGint` | A number >= 0 |
{:.rwd-table .rwd-tableFontParameters}

---
{:.hrlight}

### Add/Modify Glyphs in Fonts [11.4.4]

Applications are responsible for destroying path or image objects they have
assigned as font glyphs. It is recommended that applications destroy the path or image using `vgDestroyPath` or `vgDestroyImage` immediately after setting the object as a glyph. Since path and image objects are reference counted, destroying the object will mark its handle as invalid while leaving the resource available to the `VGFont` object.

```c
void vgSetGlyphToPath(VGFont font,
                      VGuint glyphIndex,
                      VGPath path,
                      VGboolean inHinted,
                      const VGfloat gliphOrigin[2],
                      const VGfloat escapement[2])
```

Creates a new glyph and assigns the given path to a glyph associated with the `glyphIndex` in a font object.  
The `glyphOrigin` argument defines the coordinates of the glyph origin within the `path`, and the `escapement` parameter determines the advance width for this glyph.  
Both `glyphOrigin` and `escapement` coordinates are defined in the same coordinate system as the path.  
For glyphs that have no visual representation (e.g., the space character), a value of `VG_INVALID_HANDLE` is used for path.

---
{:.hrlight}

```c
void vgSetGlyphToImage(VGFont font,
                       VGuint glyphIndex,
                       VGImage image,
                       const VGfloat gliphOrigin[2],
                       const VGfloat escapement[2])
```

Creates a new glyph and assigns the given image into a glyph associated with the `glyphIndex` in a font object.
The `glyphOrigin` argument defines the coordinates of the glyph origin within the image, and the `escapement` parameter determines the advance width for this glyph. Both `glyphOrigin` and `escapement` coordinates are defined in the image coordinate system.  
For glyphs that have no visual representation (e.g. the space character), a value of `VG_INVALID_HANDLE` is used for image.

---
{:.hrlight}

```c
void vgClearGlyph(VGFont font, VGuint glyphIndex)
```

Delete the glyph defined by a `glyphIndex` parameter from a `font`.  
The reference count for the `VGPath` or `VGImage` object to which the glyph was previously set is decremented, and the object's resources are released if the count has fallen to 0.

---

## Draw Text [11.5]

```c
void vgDrawGlyph(VGFont font,
                 VGuint glyphIndex,
                 VGbitfield paintModes,
                 VGboolean allowAutoHinting)
```

Render a glyph defined by the `glyphIndex` using the given `font` object.  
The user space position of the glyph (the point where the glyph origin will be placed) is determined by value of `VG_GLYPH_ORIGIN`.  
The new text origin is calculated by translating the glyph origin by the escapement vector of the glyph defined by `glyphIndex`.  
Following the call, the `VG_GLYPH_ORIGIN` parameter will be updated with the new origin. The `paintModes` parameter controls how glyphs are rendered.  
If paintModes is 0, neither `VGImage`-based nor `VGPath`-based glyphs are drawn.  
This mode is useful for determining the metrics of the glyph sequence.

---
{:.hrlight}

```c
void vgDrawGlyphs(VGFont font,
                  VGint glyphCount,
                  const VGuint* glyphIndices,
                  const VGfloat* adjustments_x,
                  const VGfloat* adjustments_y,
                  VGbitfield paintModes,
                  VGboolean allowAutoHinting)
```

Render a sequence of glyphs defined by the array pointed to by `glyphIndices` using the given font object.  
The values in the `adjustments_x` and `adjustments_y` arrays define positional adjustment values for each pair of glyphs defined by the `glyphIndices` array. The `glyphCount` parameter defines the number of elements in the `glyphIndices` and `adjustments_x` and `adjustments_y` arrays.  
The adjustment values defined in these arrays may represent kerning or other positional adjustments required for each pair of glyphs.  
If no adjustments for glyph positioning in a particular axis are required, `NULL` pointers may be passed for either or both of `adjustment_x` and `adjustment_y`.  
The adjustments values should be defined in the same coordinate system as the font glyphs. The user space position of the first glyph is determined by the value of `VG_GLYPH_ORIGIN`.  
A new glyph origin for every glyph in the `glyphIndices` array is calculated by translating the glyph origin by the escapement vector of the current glyph, and applying the necessary positional adjustments, taking into account both the escapement values associated with the glyphs as well as the `adjustments_x` and `adjustments_y` parameters.  
Following the call, the `VG_GLYPH_ORIGIN` parameter will be updated with the new origin.

---
