---
layout: default
title: "Configuration file"
date: 2018-11-01 08:00:00 +0100
chapter: 5
categories: [desc]
headline: "AmanithVG configuration file for rendering quality and performance fine tuning"
image: "amanithvg-logo.png"
keywords: "amanithvg configuration file threshold customize fine tuning quality performance openvg"
---

# Configuration file

**WARNING** Starting from version 5.0.3 configuration file support has been removed, in favor of two API functions:

```c
VGErrorCode vgConfigSetMZT(VGConfigMzt config,
                           VGfloat value);
```

Configure parameters and thresholds for the AmanithVG library. This function can be called at any time, but it will have effect only if:
- the library has not been already initialized by a previous call to `vgInitializeMZT`, or
- the library has been already initialized (i.e. `vgInitializeMZT` has been already called) but no contexts have been already created

```c
VGErrorCode vgConfigGetMZT(VGConfigMzt config);
```

Get the current value relative to the specified configuration parameter. If the given parameter is invalid (i.e. it does not correspond to any value of the `VGConfigMzt` enum type), a negative number is returned.

Please note that both functions do not set the internal context error, so they have no influence on the value returned by `vgGetError`.
The `VGConfigMzt` enum type defines all the parameters that can be modified:

```c
typedef enum {

    // [Geometry]
    //
    // Used by AmanithVG geometric kernel to approximate curves with straight line
    // segments(flattening). Valid range is [0; 100], where 100 represents the
    // best quality.
    VG_CONFIG_CURVES_QUALITY_MZT,
    // Used by radial gradient paints, only in non-shader pipelines (AmanithVG GLE
    // only). Valid range is [0; 100], where 100 represents the best quality.
    VG_CONFIG_RADIAL_GRADIENTS_QUALITY_MZT,
    // Used by conical gradient paints, only in non-shader pipelines (AmanithVG GLE
    // only). If VG_MZT_conical_gradient extension is not available, this parameter
    // has no effects. Valid range is [0; 100], where 100 represents the best quality.
    VG_CONFIG_CONICAL_GRADIENTS_QUALITY_MZT,

    // [Memory]
    //
    // Number of OpenVG calls (handles creation / destruction and drawing functions)
    // to be done before to recover / retrieve unused memory. Must be a positive
    // number. If 0 is specified, unused memory will never be recovered from internal
    // structures and memory pools.
    VG_CONFIG_CALLS_BEFORE_MEMORY_RECOVERY_MZT,

    // [Rasterizer cache] - AmanithVG SRE only
    //
    // Disable paths rasterizer caching if (screen space) bounding box width exceeds
    // this value. Valid values are in the range [0; 4096]. Use 0 to disable rasterizer
    // caching.
    VG_CONFIG_MAX_CACHING_BOX_WIDTH_MZT,
    // Disable paths rasterizer caching if (screen space) bounding box height exceeds
    // this value.
    // Valid values are in the range [0; 4096]. Use 0 to disable rasterizer caching.
    VG_CONFIG_MAX_CACHING_BOX_HEIGHT_MZT,
    // Allow the use of rasterizer cache forcing the vertical (screen space) bounding
    // box pixel alignement, when drawing a filled path already cached, using
    // VG_RENDERING_QUALITY_BETTER. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_ALLOW_VERTICAL_SNAP_TO_PIXEL_BETTER_MZT,
    // Allow the use of rasterizer cache forcing the vertical (screen space) bounding
    // box pixel alignement, when drawing a filled path already cached, using
    // VG_RENDERING_QUALITY_FASTER. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_ALLOW_VERTICAL_SNAP_TO_PIXEL_FASTER_MZT,
    // Allow the use of rasterizer cache forcing the vertical(screen space) bounding
    // box pixel alignement, when drawing a filled path already cached, using
    // VG_RENDERING_QUALITY_NONANTIALIASED. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_ALLOW_VERTICAL_SNAP_TO_PIXEL_NOAA_MZT,

    // [OpenGL] / [OpenGL ES] - AmanithVG GLE only
    //
    // Avoid the use of GL_EXT_texture_rectangle or GL_ARB_texture_rectangle extension,
    // even if supported by the GL Graphic System. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_RECT_TEXTURES_DISABLED_MZT,
    // Avoid the use of GL_ARB_texture_mirrored_repeat extension, even if supported
    // by the GL Graphic System. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_MIRRORED_REPEAT_DISABLED_MZT,
    // Avoid the use of GL_ARB_texture_border_clamp extension, even if supported by
    // the GL Graphic System. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_CLAMP_TO_BORDER_DISABLED_MZT,
    // Avoid the use of GL_EXT_blend_minmax extension, even if supported by the
    // GL Graphic System. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_BLEND_MIN_MAX_DISABLED_MZT,
    // Avoid the use of GL_EXT_texture_env_dot3 or GL_ARB_texture_env_dot3 extension,
    // even if supported by the GL Graphic System.
    // WARNING: setting this parameter as true will compromise the correct drawing of
    // images in stencil image mode. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_DOT3_DISABLED_MZT,
    // Avoid the use of Vertex Buffer Objects (VBO), even if supported by the
    // GL Graphic System. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_VBO_DISABLED_MZT,
    // Force the maximum number of texture units that AmanithVG can use.
    // Valid values are 1, 2, 3, 4 (AmanithVG uses no more than 4 texture units;
    // at least 2 texture units are always required to implement the whole OpenVG
    // features set).
    VG_CONFIG_MAX_PERMITTED_TEXTURE_UNITS_MZT,
    // Force the maximum texture size that AmanithVG can use.
    // Valid values are 0 (autodetect), 64, 128, 256, 512, 1024, 2048, 4096, 8192.
    // Other values will be ignored.
    VG_CONFIG_MAX_TEXTURE_SIZE_MZT,
    // When both depth and stencil buffers are available on the GL context, it forces
    // the specified buffer to be unused by AmanithVG. Valid values are defined by
    // the VGBuffersDisabledMzt enum type. Other values will be ignored.
    VG_CONFIG_FORCE_BUFFERS_DISABLED_MZT,
    // Suppose depth and stencil buffers to be persistent after a swapBuffers call.
    // Please note that, while on desktop platforms persistent buffers are common,
    // the same is not so common on embedded (OpenGL ES) platforms.
    // WARNING: setting this parameter as true on GL Graphic System with non-persistent
    // buffers, will compromise a correct rendering on some specific OpenVG features.
    // Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_SUPPOSE_PERSISTENT_BUFFERS_MZT,
    // Avoid the use of GL scissor feature, even if supported by the GL Graphic System.
    // WARNING: this parameter is provided to address compatibility issues; setting this
    // parameter as true when the stencil buffer is not available to AmanithVG, will
    // compromise a correct rendering on some specific OpenVG features.
    // Furthermore it will have a negative impact on performance.
    // Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_SCISSOR_DISABLED_MZT,
    // Avoid the use of GL color masking feature, even if supported by the
    // GL Graphic System.
    // WARNING: this parameter is provided to address compatibility issues;
    // setting this parameter as true will compromise a correct rendering on
    // some specific OpenVG features. Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_COLOR_MASKING_DISABLED_MZT,
    // Force the use of mipmaps on gradient textures.
    // Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_MIPMAPS_ON_GRADIENTS_MZT,
    // Force dithering on gradient textures, when the drawing surface is configured
    // to have less than 8bit per color component (e.g.RGB565).
    // Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_DITHERING_ON_GRADIENTS_MZT,
    // Force dithering on image textures, when the drawing surface is configured
    // to have less than 8bit per color component (e.g.RGB565).
    // Valid values are VG_FALSE and VG_TRUE
    VG_CONFIG_FORCE_DITHERING_ON_IMAGES_MZT,
    // If VG_TRUE, it forces GL_RGBA texture format even for opaque paint / images.
    // If VG_FALSE, GL_RGB texture format for opaque paint / images will be used.
    VG_CONFIG_FORCE_RGBA_TEXTURES_MZT,
    // If different than VG_FORCE_IMAGE_TEXTURE_BORDERS_NONE_MZT, it forces the
    // upload of VGImage textures with an additional filled border. If
    // VG_FORCE_IMAGE_TEXTURE_BORDERS_NONE_MZT, images are uploaded without additional
    // borders. Negative values are treated as VG_FORCE_IMAGE_TEXTURE_BORDERS_WHOLE_MZT.
    // WARNING: this parameter is provided to address compatibility issues related
    // to u-v coordinates generation throught the GL_TEXTURE matrix, on not conformant
    // GL Graphic System. By setting this parameter as
    // VG_FORCE_IMAGE_TEXTURE_BORDERS_WHOLE_MZT or to a positive number, it will
    // require additional memory and could impact on performance (when drawing images
    // for the first time).
    // Valid values are defined by the VGForceImageTextureBordersMzt enum type.
    VG_CONFIG_FORCE_IMAGE_TEXTURE_BORDERS_MZT,
    // In conjunction with VG_CONFIG_FORCE_IMAGE_TEXTURE_BORDERS_MZT parameter, it
    // specifies how VGImage texture borders must be filled. If
    // VG_TEXTURE_BORDER_MODE_CLEAR_MZT, borders are filled with a transparent black;
    // if VG_TEXTURE_BORDER_MODE_COPY_MZT, borders are filled by duplicating pixels
    // on image edges; if VG_TEXTURE_BORDER_MODE_COPY_ZERO_ALPHA_MZT, borders are
    // filled by duplicating pixels on image edges and overriding their alpha value
    // with 0. Valid values are defined by the VGImageTextureBordersModeMzt enum type.
    VG_CONFIG_IMAGE_TEXTURE_BORDERS_MODE_MZT,

    // Standard deviation factor for Gaussian blur filter. It represents the factor
    // by which the sigma value is multiplied to obtain the amplitude (of the Gaussian
    // function) to be taken for the blur. Must be a positive number.
    // The default value of 3 guarantees a coverage of the Gaussian curve equal to 99.7%
    VG_CONFIG_FILTER_GAUSSIAN_SIGMA_FACTOR_MZT
} VGConfigMzt;
```

Valid values per the `VG_CONFIG_FORCE_BUFFERS_DISABLED_MZT` parameter are defined by the `VGBuffersDisabledMzt` enum type:

```c
typedef enum {
    // The defaul value (no buffers disabled).
    VG_BUFFERS_DISABLED_NONE_MZT,
    // When both depth and stencil buffers are available on the GL context, it
    // forces the depth buffer to be unused by AmanithVG GLE.
    VG_BUFFERS_DISABLED_DEPTH_MZT,
    // When both depth and stencil buffers are available on the GL context, it
    // forces the stencil buffer to be unused by AmanithVG GLE.
    VG_BUFFERS_DISABLED_STENCIL_MZT
} VGBuffersDisabledMzt;
```

Valid values per the `VG_CONFIG_FORCE_IMAGE_TEXTURE_BORDERS_MZT` parameter are defined by the `VGForceImageTextureBordersMzt` enum type:

```c
typedef enum {
    VG_FORCE_IMAGE_TEXTURE_BORDERS_NONE_MZT,
    VG_FORCE_IMAGE_TEXTURE_BORDERS_WHOLE_MZT,
} VGForceImageTextureBordersMzt;
```

Valid values per the `VG_CONFIG_IMAGE_TEXTURE_BORDERS_MODE_MZT` parameter are defined by the `VGImageTextureBordersModeMzt` enum type:

```c
typedef enum {
    // Borders are filled with a transparent black.
    VG_TEXTURE_BORDER_MODE_CLEAR_MZT,
    // Borders are filled by duplicating pixels on image edges.
    VG_TEXTURE_BORDER_MODE_COPY_MZT,
    // Borders are filled by duplicating pixels on image edges and
    // overriding their alpha value with 0.
    VG_TEXTURE_BORDER_MODE_COPY_ZERO_ALPHA_MZT
} VGImageTextureBordersModeMzt;
```

---

For legacy reasons, here's the documentation for AmanithVG versions before 5.0.3:

AmanithVG provides a configuration file to control and configure some internal settings and thresholds.<br>
The file must be named `amanithvg.conf` on *nix (including MacOS X) Target Platforms and `amanithvg.ini` on Windows (including Windows CE) based Target Platforms.<br>
The file must be placed in `/etc` directory on *nix (including MacOS X) Target Platforms and in the same directory containing AmanithVG dll on Windows based Target Platforms, except for Windows CE that needs the file to be located within the `\Windows` directory.<br>
AmanithVG will look for the following sections:

## [OpenGL] section

Parameters present in this section control OpenGL features used by AmanithVG GLE. In the detail:

 * `forceRectTexturesDisabled`: it forces AmanithVG GLE to avoid the use of `GL_EXT_texture_rectangle` or `GL_ARB_texture_rectangle extension`, even if it's supported by the GL Graphic System.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceMirroredRepeatDisabled`: it forces AmanithVG GLE to avoid the use of `GL_ARB_texture_mirrored_repeat` extension, even if it's supported by the GL Graphic System.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceClampToBorderDisabled`: it forces AmanithVG GLE to avoid the use of `GL_ARB_texture_border_clamp` extension, even if it's supported by the GL Graphic System.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceBlendMinMaxDisabled`: it forces AmanithVG GLE to avoid the use of `GL_EXT_blend_minmax extension`, even if supported by the GL Graphic System.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceDot3Disabled`: it forces AmanithVG GLE to avoid the use of `GL_EXT_texture_env_dot3` or `GL_ARB_texture_env_dot3` extension, even if supported by the GL Graphic System.<br>
 **WARNING**: setting this parameter as true will compromise the correct drawing of images in stencil image mode.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceVBODisabled`: it forces AmanithVG GLE to avoid the use of Vertex Buffer Objects (VBO), even if they are supported by the GL Graphic System.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `maxPermittedTextureUnits`: it forces the maximum number of texture units that AmanithVG GLE can use.<br>
 Valid values are `2, 3, 4` (AmanithVG GLE uses no more than 4 texture units; at least 2 texture units are always required). The default value is `4`.

 * `maxTextureSize`: it forces the maximum texture size that AmanithVG GLE can use.<br>
 Valid values are `0 (autodetect), 64, 128, 256, 512, 1024, 2048, 4096, 8192`. Other values will be ignored. The default value is `0`.

 * `forceBufferDisabled`: when both depth and stencil buffers are available on the GL context, it forces the specified buffer to be unused by AmanithVG GLE.<br>
 Valid values are `depth`, `stencil`, `none`. Other values will be ignored. The default value is `none`.

 * `supposePersistentBuffers`: suppose depth and stencil buffers to be persistent. Please note that, while on desktop platforms persistent buffers are common, the same is not so common on embedded (OpenGL ES) platforms.<br>
 **WARNING**: setting this parameter as true on GL Graphic System with non-persistent buffers, will compromise a correct rendering on some specific OpenVG features.<br>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceScissorDisabled`: avoid the use of GL scissor feature, even if supported by the GL Graphic System.<br> **WARNING**: this parameter is provided to address compatibility issues; setting this parameter as `true` when the stencil buffer is not available to AmanithVG GLE, will compromise a correct rendering on some specific OpenVG features. Furthermore it will have a negative impact on performance.<br>
 Valid values are `false` and `true`. The default is `false`.

 * `forceColorMaskingDisabled`: avoid the use of GL color masking feature, even if supported by the GL Graphic System.<br> **WARNING**: this parameter is provided to address compatibility issues; setting this parameter as `true` will compromise a correct rendering on some specific OpenVG features.<br>
 Valid values are `false` and `true`. The default is `false`.

 * `forceMipMapsOnGradients`: it forces AmanithVG GLE to use mipmaps on gradient textures.<br>
 Valid values are `false` and `true`. The default is `false`.

 * `forceDitheringOnGradients`: it forces AmanithVG GLE to dither gradient textures, when the drawing surface is configured to have less than 8bit per color component (e.g. RGB565).<br>
 Valid values are `false` and `true`. The default is `false`.

 * `forceDitheringOnImages`: it forces AmanithVG GLE to dither image textures, when the drawing surface is configured to have less than 8bit per color component (e.g. RGB565).<br>
 Valid values are `false` and `true`. The default is `false`.

 * `forceRGBATextures`: if true, it forces GL\_RGBA texture format even for opaque paint/images. If false use GL_RGB texture format for opaque paint/images.<br>
 Valid values are `false` and `true`. The default is `true`.

---

## [Geometry] section

Thresholds present in this section define the accuracy of AmanithVG geometry engine. In the detail:

 * `curvesQuality`: it affects the process that approximates curves with a set of straight line segments (flattening).<br>
 Valid range is `[0; 100]`, where `100` represents the best achievable quality. The default value is `90`.

 * `radialGradientsQuality`: it affects the quality of radial gradients paint.<br>
 Valid range is `[0; 100]`, where `100` represents the best achievable quality. The default value is `75`.

 * `conicalGradientsQuality`: it affects the quality of conical gradients paint, when `VG_MZT_conical_gradient` extension is available.<br>
 Valid range is `[0; 100]`, where `100` represents the best achievable quality. The default value is `75`.

---
