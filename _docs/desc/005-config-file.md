---
layout: default
title: "Configuration file"
date: 2017-01-01 08:00:00 +0100
chapter: 5
categories: [desc]
---

# Configuration file

AmanithVG provides a configuration file to control and configure some internal settings and thresholds.<br/>
The file must be named `amanithvg.conf` on *nix (including MacOS X) Target Platforms and `amanithvg.ini` on Windows (including Windows CE) based Target Platforms.<br/>
The file must be placed in `/etc` directory on *nix (including MacOS X) Target Platforms and in the same directory containing AmanithVG dll on Windows based Target Platforms, except for Windows CE that needs the file to be located within the `\Windows` directory.<br/>
AmanithVG will look for the following sections:

## [OpenGL] section

Parameters present in this section control OpenGL features used by AmanithVG GLE. In the detail:

 * `forceRectTexturesDisabled`: it forces AmanithVG GLE to avoid the use of `GL_EXT_texture_rectangle` or `GL_ARB_texture_rectangle extension`, even if it's supported by the GL Graphic System.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceMirroredRepeatDisabled`: it forces AmanithVG GLE to avoid the use of `GL_ARB_texture_mirrored_repeat` extension, even if it's supported by the GL Graphic System.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceClampToBorderDisabled`: it forces AmanithVG GLE to avoid the use of `GL_ARB_texture_border_clamp` extension, even if it's supported by the GL Graphic System.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceBlendMinMaxDisabled`: it forces AmanithVG GLE to avoid the use of `GL_EXT_blend_minmax extension`, even if supported by the GL Graphic System.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceDot3Disabled`: it forces AmanithVG GLE to avoid the use of `GL_EXT_texture_env_dot3` or `GL_ARB_texture_env_dot3` extension, even if supported by the GL Graphic System.<br/>
 **WARNING**: setting this parameter as true will compromise the correct drawing of images in stencil image mode.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceVBODisabled`: it forces AmanithVG GLE to avoid the use of Vertex Buffer Objects (VBO), even if they are supported by the GL Graphic System.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `maxPermittedTextureUnits`: it forces the maximum number of texture units that AmanithVG GLE can use.<br/>
 Valid values are `2, 3, 4` (AmanithVG GLE uses no more than 4 texture units; at least 2 texture units are always required). The default value is `4`.

 * `maxTextureSize`: it forces the maximum texture size that AmanithVG GLE can use.<br/>
 Valid values are `0 (autodetect), 64, 128, 256, 512, 1024, 2048, 4096, 8192`. Other values will be ignored. The default value is `0`.

 * `forceBufferDisabled`: when both depth and stencil buffers are available on the GL context, it forces the specified buffer to be unused by AmanithVG GLE.<br/>
 Valid values are `depth`, `stencil`, `none`. Other values will be ignored. The default value is `none`.

 * `supposePersistentBuffers`: suppose depth and stencil buffers to be persistent. Please note that, while on desktop platforms persistent buffers are common, the same is not so common on embedded (OpenGL ES) platforms.<br/>
 **WARNING**: setting this parameter as true on GL Graphic System with non-persistent buffers, will compromise a correct rendering on some specific OpenVG features.<br/>
 Valid values are `false` and `true`. The default value is `false`.

 * `forceScissorDisabled`: avoid the use of GL scissor feature, even if supported by the GL Graphic System.<br/> **WARNING**: this parameter is provided to address compatibility issues; setting this parameter as `true` when the stencil buffer is not available to AmanithVG GLE, will compromise a correct rendering on some specific OpenVG features. Furthermore it will have a negative impact on performance.<br/>
 Valid values are `false` and `true`. The default is `false`.

 * `forceColorMaskingDisabled`: avoid the use of GL color masking feature, even if supported by the GL Graphic System.<br/> **WARNING**: this parameter is provided to address compatibility issues; setting this parameter as `true` will compromise a correct rendering on some specific OpenVG features.<br/>
 Valid values are `false` and `true`. The default is `false`.

 * `forceMipMapsOnGradients`: it forces AmanithVG GLE to use mipmaps on gradient textures.<br/>
 Valid values are `false` and `true`. The default is `false`.

 * `forceDitheringOnGradients`: it forces AmanithVG GLE to dither gradient textures, when the drawing surface is configured to have less than 8bit per color component (e.g. RGB565).<br/>
 Valid values are `false` and `true`. The default is `false`.

 * `forceDitheringOnImages`: it forces AmanithVG GLE to dither image textures, when the drawing surface is configured to have less than 8bit per color component (e.g. RGB565).<br/>
 Valid values are `false` and `true`. The default is `false`.

 * `forceRGBATextures`: if true, it forces GL\_RGBA texture format even for opaque paint/images. If false use GL_RGB texture format for opaque paint/images.<br/>
 Valid values are `false` and `true`. The default is `true`.

---

## [Geometry] section

Thresholds present in this section define the accuracy of AmanithVG geometry engine. In the detail:

 * `curvesQuality`: it affects the process that approximates curves with a set of straight line segments (flattening).<br/>
 Valid range is `[0; 100]`, where `100` represents the best achievable quality. The default value is `90`.

 * `radialGradientsQuality`: it affects the quality of radial gradients paint.<br/>
 Valid range is `[0; 100]`, where `100` represents the best achievable quality. The default value is `75`.

 * `conicalGradientsQuality`: it affects the quality of conical gradients paint, when `VG_MZT_conical_gradient` extension is available.<br/>
 Valid range is `[0; 100]`, where `100` represents the best achievable quality. The default value is `75`.

---
