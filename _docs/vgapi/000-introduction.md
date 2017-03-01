---
layout: default
title: "Introduction"
date: 2017-01-01 08:00:00 +0100
chapter: 0
categories: [vgapi]
---

# Introduction

OpenVG is an application programming interface (API) for hardware-accelerated two-dimensional vector and raster graphics, it provides a device-independent and vendor-neutral interface for sophisticated 2D graphical applications.

OpenVG provides a drawing model similar to those of existing two-dimensional drawing APIs and formats, such as Adobe PostScript, PDF, Adobe Flash, Sun Microsystems Java2D and W3C SVG. OpenVG version 1.1 is specifically intended to support all drawing features required by a SVG Tiny 1.2 renderer or an Adobe Flash Lite renderer, and additionally to support functions that may be of use for implementing an SVG Basic renderer.

## Target Applications

 * __SVG Viewers__   
 OpenVG must provide the drawing functionality required for a high performance SVG document viewer that is conformant with version 1.2 of the SVG Tiny profile. It does not need to provide a one-to-one mapping between SVG syntactic features and API calls, but it must provide efficient ways of implementing all SVG Tiny features.

 * __Portable Mapping Applications__   
 OpenVG can provide dynamic features for map display that would be difficult or impossible to do with an SVG viewer alone, such as dynamic placement and sizing of street names and markers, and efficient viewport culling.

 * __E-book Readers__  
 The OpenVG API must provide fast rendering of readable text in Western, Asian, and other scripts. It does not need to provide advanced text layout features.

 * __Games__  
 The OpenVG API must be useful for defining sprites, backgrounds, and textures for use in both 2D and 3D games. It must be able to provide twodimensional overlays (e.g., for maps or scores) on top of 3D content.

 * __Scalable User Interfaces__  
 OpenVG may be used to render scalable user interfaces, particularly for applications that wish to present users with a unique look and feel that is consistent across different screen resolutions.
 
 * __Low-Level Graphics Device Interface__   
 OpenVG may be used as a low-level graphics device interface. Other graphical toolkits, such as windowing systems, may be implemented above OpenVG.


## OpenVG API Design Philosphy

 * __Simplicity__ subtracting the obvious and adding the meaningful (help developer "guess the answer").

 * __OpenGL-style syntax__ is used where possible, in order to make learning OpenVG as easy as possible for OpenGL developers.

 * __Extensibility__ makes it possible to add new state variables in order to add new features to the pipeline without needing to add new functions.

 * __Focus__ on embedded devices* like mobile phone, PDA, game console, DVR, DVD, car navigation, etc.

---
