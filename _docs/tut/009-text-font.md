---
layout: default
title: "Tut. 09 - Text & Fonts"
date: 2018-11-01 08:00:00 +0100
chapter: 9
categories: [tut]
headline: "AmanithVG, how to deal with fonts and text drawing functions"
image: "tut09_screenshot_dark.png"
keywords: "amanithvg tutorial 09 draw text font glyph layout kerning escapement openvg"
---

# Tutorial 9 : Text & Fonts

Several classes of applications were considered in order to determining the set of features supported by the OpenVG text rendering API: e-book readers, scalable user interfaces with text-driven menus, SVG viewers and games.
These application requirements made it clear that OpenVG must provide a fast, low-level API that is capable of supporting vector glyph outlines, as well as glyphs represented as bitmaps.

The process of text rendering involves the following steps:

 - selection of a font, font style and size

 - scaling of glyphs used in a text fragment

 - composing the text on a page or within a text box

 - rendering of glyph outlines into bitmap images

 - blitting of bitmap images to a frame buffer or a screen.

Font and glyph scaling is usually done once for each selected text size; however, the rendering of glyph outlines and blitting of bitmaps is repeated routinely.
Let's have a look at how we can handle fonts and text rendering in OpenVG.

---

## Glyph positioning and text layout

Scalable fonts define glyphs using vector outlines and additional set of data, such as hinting instructions, font and glyph metrics, etc. A typical glyph would be defined by the following information:

- glyph origin: glyph origin is not always located at the glyph boundary; glyphs from various custom or script fonts may have swashes and ornamental design with the glyph origin located inside the bounding box

- glyph outline: the data defining vector outlines

- glyph bounding box: the bounding box that encloses vector outlines

- escapement: each scaled and rendered glyph is positioned in such a way that the current glyph origin is located at the same point that is defined by the "advance width", or escapement of the previous character

| &nbsp; |
| :---: |
| *Some basic properties of glyphs: origin, outline, bounding box, escapement* |
{:.tbl_images .tut09_glyph_metrics} 

The complexity of text rendering and composition depends on language scripts. In many simple scripts (such as western and eastern European languages) text is composed by simply planking glyphs next to each other along the horizontal baseline.
The next glyph origin must be calculated using the *escapement* for the current glyph. Note that vector defined by two points *glyph origin* and *escapement* must be subjected to the same matrix transformation that is applied to a glyph outline when the glyph is scaled.

In some cases, the text composition requires that glyph layout and positioning be adjusted along the baseline (using kerning) to account for the difference in appearance of different glyphs and to achieve uniform typographic color (optical density) of the text.

| &nbsp; | 
| :---: |
| *Glyph positioning with kerning* |
{:.tbl_images .tut09_glyph_kerning} 

Some complex scripts require glyph positioning be adjusted in both directions; therefore, when a text composition involves support for complex scripts, the inter-character spacing between each pair of glyphs in a text string may have to be defined using the *escapement* for the current glyph (i), and the additional *adjustment* vector for the pair of glyphs (i, i + 1).

---

## OpenVG text

OpenVG provides a mechanism to allow applications to define a `VGFont` object as a collection of glyphs, where each glyph can be represented as either a `VGPath` representing either an original unhinted outline that can be scaled and rendered, or a scaled and hinted outline; or a `VGImage` representing a scaled, optionally hinted, and rendered image of a glyph. Use of a `VGImage` provides the opportunity to satisfy rendering quality requirements that cannot be achieved by generic outline rendering. No further hinting is applied to image glyphs.

`VGFont` objects are created by an application, and can contain either a full set of glyphs or a subset of glyphs of an original font. `VGFont` objects do not contain any metric or layout information; instead, applications are responsible for all text layout operations using the information provided by the original fonts. In other words, OpenVG can assist applications in text composition by providing glyph positioning calculations; however, the **text layout is the responsibility of the application**.

Glyphs in a `VGFont` are identified by a *glyph index*, which is an arbitrary number assigned to a glyph when it is created. This mapping mechanism is similar to the glyph mapping used in standard font formats, such as TrueType or OpenType fonts, where each glyph is assigned an index that is mapped to a particular character code using a separate mapping table. The semantics of the mapping are application-dependent, possible mappings include:

 - Unicode character codes: when a `VGFont` is created as a subset that supports only simple language scripts (e.g., Latin, with simple one-to-one character-toglyph mapping), the character code values may be used as glyph indices. This eliminates the need for an additional mapping table and simplifies text rendering (a text string may be passed directly as an argument to OpenVG API call for text rendering).

 - Native font glyph indices: OpenVG applications may re-use native glyph indices from an original TrueType or OpenType font when `VGFont` object is created; this simplifies text composition and layout decisions by re-using OpenType/TrueType layout and character-to-glyph mapping tables (and any platform-supplied text composition engine).
 
 - Application-defined (custom) glyph indices: OpenVG applications may assign arbitrary numbers as glyph indices. This may be beneficial for special purpose fonts that have a limited number of glyphs (e.g. SVG fonts).

`VGFont` objects are created and destroyed using the `vgCreateFont` and `vgDestroyFont` functions. Font glyphs may be added (`vgSetGlyphToPath`, `vgSetGlyphToImage`), replaced or deleted (`vgClearGlyph`) after the font has been created.
Please refer to the ["Text and font" section]({{site.url}}/docs/vgapi/012-text-and-font.html) for the detailed API description.
The following example (adapted from the tutorial code) shows a possible vector glyph definition structure and a classic OpenVG font creation and initialization function.

```c
typedef struct {
    // glyph index within the OpenVG font object
    VGuint glyphIndex;
    // glyph origin
    VGfloat glyphOrigin[2];
    // the advance for this glyph
    VGfloat escapement[2];
    // OpenVG path commands defining glyph outline
    VGint commandsCount;
    VGubyte* commands;
    // OpenVG path coordinates defining glyph outline
    VGint coordinatesCount;
    VGfloat* coordinates;
} Glyph;

typedef struct {
    // OpenVG font object
    VGFont openvgHandle;
    // number of glyphs
    VGuint glyphsCount;
    // glyphs array
    Glyph* glyphs;
} Font;

// initialize a given font
void fontInit(Font* font) {

    VGuint i;

    // create OpenVG font object
    font->openvgHandle = vgCreateFont(font->glyphsCount);
    // create OpenVG glyphs
    for (i = 0; i < font->glyphsCount; ++i) {
        // specify segments and coordinates capacity hint, so the OpenVG driver
        // could try to optimize memory allocation
        VGPath path = vgCreatePath(VG_PATH_FORMAT_STANDARD, VG_PATH_DATATYPE_F,
                                   1.0f, 0.0f, font->glyphs[i].commandsCount,
                                   font->glyphs[i].coordinatesCount,
                                   VG_PATH_CAPABILITY_ALL);
        // upload vector data (i.e. glyph outlines)
        vgAppendPathData(path, font->glyphs[i].commandsCount, font->glyphs[i].commands,
                         font->glyphs[i].coordinates);
        // remove "editing" capabilities, so that OpenVG driver
        // can try to free some memory
        vgRemovePathCapabilities(path, VG_PATH_CAPABILITY_APPEND_FROM |
                                       VG_PATH_CAPABILITY_APPEND_TO |
                                       VG_PATH_CAPABILITY_MODIFY | 
                                       VG_PATH_CAPABILITY_TRANSFORM_FROM |
                                       VG_PATH_CAPABILITY_TRANSFORM_TO | 
                                       VG_PATH_CAPABILITY_INTERPOLATE_FROM |
                                       VG_PATH_CAPABILITY_INTERPOLATE_TO);
        // associate the created path to the given glyph index
        vgSetGlyphToPath(font->openvgHandle, font->glyphs[i].glyphIndex, path, VG_FALSE,
                         font->glyphs[i].glyphOrigin, font->glyphs[i].escapement);
        // since path objects are reference counted, destroying the object will mark its
        // handle as invalid while leaving the resource available to the VGFont object
        vgDestroyPath(path);
    }
}

// destroy a given font
void fontDestroy(Font* font) {
    // destroying the VGFont object will destroy/release all previously attached
    // paths, because within the fontInit function we already dereferenced them
    // through the vgDestroyPath call
    vgDestroyFont(font->openvgHandle);
}
```

---

## The tutorial code

The tutorial draws two text strings along two different trajectories: one straight line and a curve trait.
The straight line trajectory is defined by two control points, that can be moved using the mouse/touch. The wavy trajectory is defined by a single cubic Bézier curve, so four control points are needed.

| &nbsp; |
| :---: |
| *The text routes: a line and a cubic Bézier curve* |
{:.tbl_images .tut09_trajectories} 

In both cases, the text string must cover the whole trajectory, so we must implement:

 1. a function that calculates the text string length in glyph/font units, taking care of escapements and kerning

 2. a function that calculates the trajectory length in drawing surface space (control points are expressed in the drawing surface reference system)

 3. a function that calculates a font size so that the text string will span the whole route

 4. a function that draws the text string following the trajectory baseline

Lets start with the first function. In order to calculate a text string length in glyph/font units, we take advantage of the OpenVG `vgDrawGlyphs` function: such function is normally used to draw a set of glyphs, but under some conditions can be used to extract the metric we are looking for.

```c
void vgDrawGlyphs(VGFont font,
                  VGint glyphCount,
                  const VGuint* glyphIndices,
                  const VGfloat* adjustments_x,
                  const VGfloat* adjustments_y,
                  VGbitfield paintModes,
                  VGboolean allowAutoHinting)
````

The `paintModes` parameter controls how glyphs are rendered. If `paintModes` is `0`, neither glyphs are drawn: this mode is useful for determining the metrics of the glyph sequence.
The user space position of the glyph (i.e. the point where the glyph origin will be placed) is determined by value of `VG_GLYPH_ORIGIN`; such OpenVG context parameter is updated after drawing each glyph of sequence of glyphs.
So, the function (1) will look like the following one (see `font_common.c` file):

```c
// temporary arrays, used by the textLineBuild function
VGuint* glyphIndices;
VGfloat* adjustmentsX;
VGfloat* adjustmentsY;

void textLineBuild(Font* font,
                   char* str,
                   size_t strLen) {

    size_t i;
    VGint leftGlyphIndex = glyphIndexFromCharCode(font, str[0]);

    // first glyph index
    glyphIndices[0] = leftGlyphIndex;
    for (i = 1; i < strLen; ++i) {
        VGint rightGlyphIndex = glyphIndexFromCharCode(font, str[i]);
        // get kerning info relative to the (leftGlyphIndex, rightGlyphIndex) couple
        KerningEntry* krn = kerningFromGlyphIndices(font,
                                                    leftGlyphIndex, rightGlyphIndex);
        // append glyph index
        glyphIndices[i] = rightGlyphIndex;
        // initialize adjustment
        adjustmentsX[i - 1] = 0.0f;
        adjustmentsY[i - 1] = 0.0f;
        // add kerning info
        if (krn != NULL) {
            adjustmentsX[i - 1] += krn->x;
            adjustmentsY[i - 1] += krn->y;
        }
        leftGlyphIndex = rightGlyphIndex;
    }
    // last adjustment entry
    adjustmentsX[strLen - 1] = 0.0f;
    adjustmentsY[strLen - 1] = 0.0f;
}

// get text string width, in glyph/font space
VGfloat textLineWidth(Font* font,
                      char* str) {

    // get number of characters
    size_t strLen = strlen(str);
    // start at origin (just a convention)
    VGfloat glyphOrigin[2] = { 0.0f, 0.0f };

    // build the sequence of glyph indices and kerning data
    textLineBuild(font, str, strLen);
    // calculate the metrics of the glyph sequence
    vgSetfv(VG_GLYPH_ORIGIN, 2, glyphOrigin);
    vgSeti(VG_MATRIX_MODE, VG_MATRIX_GLYPH_USER_TO_SURFACE);
    vgLoadIdentity();
    vgDrawGlyphs(font->openvgHandle, strLen, glyphIndices,
    	         adjustmentsX, adjustmentsY, 0, VG_FALSE);
    vgGetfv(VG_GLYPH_ORIGIN, 2, glyphOrigin);
    return glyphOrigin[0];
}
```

Now we take a look at the function (2), the calculation of trajectories length (surface space). For the straight line trajectory is really simple, it's a matter to calculate a distance between two 2D points.
For the curved trajectory it is simple too, we just need to invoke the OpenVG `vgPathLength` function, that calculates the lenght of a given path (in our case the wavy path).
Function (3) is really simple: in order to calculate a font size so that a text string will span a whole trajectory, we simply divide the trajectory length by the text string length.

```c
// straight text parameters
VGfloat straightTextFontSize;
char* straightText = "AmanithVG engine";

// wavy text parameters
VGfloat wavyTextFontSize;
char* wavyText = "OpenVG text example";

void fontSizeCalculate(void) {

    VGfloat objSpaceWidth, srfSpaceLen;

    // calculate the text width, in glyph/font space
    objSpaceWidth = textLineWidth(&font, straightText);
    // calculate the text width, in surface space
    srfSpaceLen = distance(controlPoints[0], controlPoints[1]);
    // calculate font size
    straightTextFontSize = srfSpaceLen / objSpaceWidth;
    
    // calculate the text width, in glyph/font space
    objSpaceWidth = textLineWidth(&font, wavyText);
    // calculate the text width, in surface space
    srfSpaceLen = vgPathLength(textCurve, 0,
    	                       vgGetParameteri(textCurve, VG_PATH_NUM_SEGMENTS));
    // calculate font size
    wavyTextFontSize = srfSpaceLen / objSpaceWidth;
}
```

Function (4) implements the drawing of a text string along a trajectory. The function has been actually split in two functions, one for the straight line trajectory, and one for the curved trajectory.
The first one makes use of the `vgDrawGlyphs` function, in order to draw a set of given glyph indices and kerning adjustments:

```c
// draw a line of text
void textLineDraw(Font* font,
                  char* str,
                  VGbitfield paintModes) {

    size_t strLen = strlen(str);
    // build the sequence of glyph indices and kerning data
    textLineBuild(font, str, strLen);
    // draw glyphs
    vgDrawGlyphs(font->openvgHandle, strLen, glyphIndices,
    	         adjustmentsX, adjustmentsY, paintModes, VG_FALSE);
}

void straightTextDraw(void) {

    VGfloat glyphOrigin[2] = { 0.0f, 0.0f };
    // calculate text rotation (radians)
    VGfloat rotation = atan2(controlPoints[1].y - controlPoints[0].y,
                             controlPoints[1].x - controlPoints[0].x);

    vgSetfv(VG_GLYPH_ORIGIN, 2, glyphOrigin);
    vgSeti(VG_MATRIX_MODE, VG_MATRIX_GLYPH_USER_TO_SURFACE);
    vgLoadIdentity();
    vgTranslate(controlPoints[0].x, controlPoints[0].y);
    // rotation in degrees
    vgRotate(rotation * 57.2957795f);
    // set font size
    vgScale(straightTextFontSize, straightTextFontSize);
    // draw straight text
    textLineDraw(&font, straightText, VG_FILL_PATH);
}
```

| &nbsp; |
| :---: |
| *Text follows the straight line route* |
{:.tbl_images .tut09_straight_text} 

In order to draw a text along a wavy path, we cannot use the same `vgDrawGlyphs` function that we have used before, because we must change the `VG_MATRIX_GLYPH_USER_TO_SURFACE` matrix for every glyph, in order to match path position and slope. So we must stick to the `vgDrawGlyph` function (that draws a single glyph) with the additional help of `vgPointAlongPath` function in order to calculate, for each glyph, path position and slope.

```c
// draw a text along the given path
void textAlongPathDraw(Font* font,
                       VGPath path,
                       char* str,
                       VGfloat fontSize,
                       VGbitfield paintModes) {

    size_t i, strLen = strlen(str);
    VGfloat cursor = 0.0f;
    // set glypth origin to (0, 0)
    VGfloat glyphOrigin[2] = { 0.0f, 0.0f };
    VGint pathSegmentsCount = vgGetParameteri(path, VG_PATH_NUM_SEGMENTS);

    // build the sequence of glyph indices and kerning data
    textLineBuild(font, str, strLen);

    // loop over all characters
    for (i = 0; i < strLen; ++i) {
        VGfloat x, y, tgx, tgy;
        // get glyph
        Glyph* glyph = glyphFromGlyphIndex(font, glyphIndices[i]);
        VGfloat halfEscapement = glyph->escapement[0] * 0.5f;

        // evaluate path
        cursor += halfEscapement;
        vgPointAlongPath(path, 0, pathSegmentsCount, cursor * fontSize,
                         &x, &y, &tgx, &tgy);
        // set matrix in order to match path position and slope
        vgSetfv(VG_GLYPH_ORIGIN, 2, glyphOrigin);
        vgSeti(VG_MATRIX_MODE, VG_MATRIX_GLYPH_USER_TO_SURFACE);
        vgLoadIdentity();
        vgTranslate(x, y);
        vgRotate(atan2(tgy, tgx) * 57.2957795f);
        // set font size
        vgScale(fontSize, fontSize);
        vgTranslate(-halfEscapement, 0.0f);
        // draw glyph
        vgDrawGlyph(font->openvgHandle, glyphIndices[i], paintModes, VG_FALSE);
        // advance cursor
        cursor += halfEscapement + adjustmentsX[i];
    }
}
```

| &nbsp; |
| :---: |
| *Text along a wavy path* |
{:.tbl_images .tut09_wavy_text} 

A note about the actual Font structure defined within the code. The real structure contains two important tables: *character codes map* and *kerning table*.
The first is a simple array of the following structure (see `font_common.h` file):

```c
typedef struct {
    VGuint charCode;
    VGuint glyphIndex;
} MappedChar;
```

Each entry maps a character code to a glyph index; the map is sorted by ascending character codes, so a binary search could be used to accelerate queries like "given a character code, plese return me its glyph index".
If the font does not include a *character codes map*, it means that glyph indices coincide with character codes (see `font_common.c` file):

```c
// compare two mapped characters
VGint mappedCharsCompare(void* arg0,
                         void* arg1) {

    MappedChar* ch0 = (MappedChar*)arg0;
    MappedChar* ch1 = (MappedChar*)arg1;
    return (VGint)ch0->charCode - (VGint)ch1->charCode;
}

// given a character code, return its glyph index
VGint glyphIndexFromCharCode(Font* font,
                             VGint charCode) {

    VGint glyphIndex = -1;

    if (font->charCodesMap != NULL) {
        MappedChar ch = { charCode, 0 };
        // "character code" to "glyph index" map is sorted by ascending character
        // codes, so we can perform a binary search in order to speedup things
        MappedChar* found = (MappedChar*)bsearch(&ch, font->charCodesMap, 
                                                 font->charCodesMapSize,
                                                 sizeof(MappedChar),
                                                 mappedCharsCompare);
        if (found) {
            glyphIndex = found->glyphIndex;
        }
    }
    else {
        // in this case glyph indices coincide with character codes
        glyphIndex = charCode;
    }
 
    return glyphIndex;
}
```

The *kerning table* is a simple array of the following structure (see `font_common.h` file):

```c
typedef struct {
    // the key representing two glyph indices ((leftGlyphIndex << 16) + rightGlyphIndex)
    VGuint key;
    // the kerning amount relative to the glyphs couple
    VGfloat x;
    VGfloat y;
} KerningEntry;
```

Each entry maps a glyph indices couple to the corresponding (non-zero) kerning value; table is sorted by ascending glyph indices, so a binary search could be used to accelerate queries like "given two glyph indices (representing two characters), plese return me the kerning value". If the font does not include a *kerning table*, it means that kerning is always 0, regardless of the glyph indices:

```c
// compare two kerning entries
VGint kerningsCompare(void* arg0,
                      void* arg1) {

    KerningEntry* krn0 = (KerningEntry*)arg0;
    KerningEntry* krn1 = (KerningEntry*)arg1;
    return (VGint)krn0->key - (VGint)krn1->key;
}

// given a couple of glyph indices, return the relative
// kerning data (NULL if kerning is zero)
KerningEntry* kerningFromGlyphIndices(Font* font,
                                      VGint leftGlyphIndex,
                                      VGint rightGlyphIndex) {

    KerningEntry krn = {
        // the key
        ((VGuint)leftGlyphIndex << 16) + (VGuint)rightGlyphIndex,
        0.0f, 0.0f
    };

    return (KerningEntry*)bsearch(&krn, font->kerningTable,
    	                          font->kerningTableSize, sizeof(KerningEntry),
    	                          kerningsCompare);
}
```

| &nbsp; |
| :---: |
| *The full tutorial example* |
{:.tbl_images .tut09_screenshot} 

---