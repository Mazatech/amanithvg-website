---
layout: default
title: "Tut. 01 - Common setup"
date: 2017-01-01 08:00:00 +0100
chapter: 1
categories: [tut]
---

# Tutorial 01: Common Setup

This tutorial will look at some basic OpenVG concepts, especially about the platform setup.

OpenVG functions that perform drawing, or that modify or query drawing state, make use of an implicit context.
OpenVG API calls are executed with respect to the context currently bound to the thread in which they are executed: it is called the *current context*.
When an image, paint, path, font, or mask handle is defined, it is permanently attached to the context that is current at that time.

So we can think at an OpenVG context as a container of:

 - drawing objects (paints, paths, images, fonts, mask layers) that can be created, modified, drawn and destroyed through the OpenVG API

 - a set of drawing states that could be queried and modified through the OpenVG API

A drawing surface is a rectangular region of pixels, where all drawing operations are performed to.

A context is created, attached to a drawing surface, and bound to a running application thread outside the scope of the OpenVG API.

AmanithVG provides a set of custom functions to create, bind and destroy OpenVG contexts and drawing surfaces; see [EGL like interface]({% link _docs/desc/003-egl.md %}) for additional details.

The most used functions are the following:

```c
// Create and initialize an OpenVG context
void* vgPrivContextCreateMZT(void* sharedContext);
```

```c
// Destroy a previously created OpenVG context
void vgPrivContextDestroyMZT(void* context);
```

```c
// Get the maximum dimension allowed for OpenVG drawing surfaces
VGint vgPrivSurfaceMaxDimensionGetMZT(void);
```

```c
// Create and initialize a drawing surface, specifying dimensions (in pixels) and format
void* vgPrivSurfaceCreateMZT(VGint width, VGint height,
                             VGboolean linearColorSpace,
                             VGboolean alphaPremultiplied,
                             VGboolean alphaMask);
```

```c
// Destroy a previously created drawing surface
void vgPrivSurfaceDestroyMZT(void* surface);
```

```c
// Get the width (in pixels) of the given drawing surface
VGint vgPrivGetSurfaceWidthMZT(const void* surface);
```

```c
// Get the height (in pixels) of the given drawing surface
VGint vgPrivGetSurfaceHeightMZT(const void* surface);
```

```c
// Bind the specified context to the given drawing surface
VGboolean vgPrivMakeCurrentMZT(void *context, void *surface);
```

So the typical OpenVG initialization (and relative cleanup) is as follow:

```c
// OpenVG variables
void* vgContext = NULL;
void* vgSurface = NULL;

void openvgInit(int width, int height) {

    // create an OpenVG context
    vgContext = vgPrivContextCreateMZT(NULL);

    // create a drawing surface (sRGBA premultiplied color space)
    vgSurface = vgPrivSurfaceCreateMZT(width, height, VG_FALSE, VG_TRUE, VG_TRUE);

    // bind context and surface
    vgPrivMakeCurrentMZT(vgContext, vgSurface);
}
```

Of course the actual code must take care of possible errors during the initialization.

```c
void openvgDestroy(void) {
    
    // unbind context and surface
    vgPrivMakeCurrentMZT(NULL, NULL);

    // destroy OpenVG surface
    vgPrivSurfaceDestroyMZT(vgSurface);

    // destroy OpenVG context
    vgPrivContextDestroyMZT(vgContext);
}
```

After such initialization, we can start to use OpenVG API to create and draw shapes, text and so on.
A typical window-based (i.e. not console) application loop, looks like the following one:

```c
// calculate a feasible window dimension: not bigger than screen nor
// bigger than maximum dimension allowed for OpenVG drawing surfaces
int screenDim = min(screenWidth(), screenHeight());
int openvgDim = vgPrivSurfaceMaxDimensionGetMZT();
int windowSize = min(screenDim, openvgDim);

// create a native square window
NativeHandle windowHandle = windowCreate(windowSize, windowSize);
if (amanithvg_gle) {
    // initialize OpenGL (ES) upon the created window
    glesInit(windowHandle);
}

// initialize OpenVG
openvgInit(windowSize, windowSize);

while (!done) {
    < OpenVG rendering code >
    if (amanithvg_gle) {
        // AmanithVG GLE: use the native OpenGL (ES) swap buffers function
        glesSwapBuffers();
    }
    else {
        // AmanithVG SRE: get OpenVG surface dimensions and pixels buffer
        int width = vgPrivGetSurfaceWidthMZT(vgSurface);
        int height = vgPrivGetSurfaceHeightMZT(vgSurface);
        char* pixels = vgPrivGetSurfacePixelsMZT(vgSurface);
        blitBuffer(windowHandle, width, height, pixels);
    }
    // acknowledge AmanithVG that we have performed a swapbuffers operation
    vgPostSwapBuffersMZT();
}

// destroy OpenVG context and surface
destroyOpenVG();

// destroy OpenGL (ES) resources
if (amanithvg_gle) {
    glesDestroy();
}

// destroy native window
windowDestroy(windowHandle);
```

Of course, some functions heavily depends on the platform windowing system: `screenWidth`, `screenHeight`, `windowCreate`, `windowDestroy`.
Some others depend on the platform OpenGL (ES) system: `glesInit`, `glesSwapBuffers`, `glesDestroy`.

The structure of `glesInit` and `glesDestroy` functions is as follow:

```c
void glesInit(NativeHandle windowHandle) {

    if (iOS) {
        < create an OpenGL (ES) context >
        < make the OpenGL (ES) context current >
        < create an OpenGL (ES) framebuffer object, attaching color, depth and stencil buffers >
        < make the framebuffer object current >
    }
    else {
        < select a configuration supporting color, depth and stencil (and maybe multisample) >
        < create an OpenGL (ES) context on top of the given windowHandle >
        < make the OpenGL (ES) context current >
    }
}
```

```c
void glesDestroy(void) {

    if (iOS) {
        < unbind the framebuffer object >
        < destroy the framebuffer object >
    }
    < unbind OpenGL (ES) context >
    < destroy OpenGL (ES) context >
}
```

A note about AmanithVG SRE and the `blitBuffer` function: AmanithVG SRE performs all the rendering in system memory.
A blitter must be written in order to copy the content of AmanithVG SRE drawing surfaces to the window/video memory.
Another viable option is to use OpenGL (ES), upload surface pixels to a GL texture and then draw a 2D textured rectangle.
Here is the list of actual API used to blit AmanithVG SRE drawing surface content:

 - Windows: SetDIBitsToDevice (GDI function)

 - MacOS X: OpenGL + texture + dedicated Apple extensions

 - Linux: XPutImage (xlib)

 - iOS: OpenGL (ES) + texture

 - Android: OpenGL (ES) + texture

---

The whole example code includes two logical subsections: the platform dependent code (window setup, keyboard/mouse/touch handlers), and the platform independent OpenVG code (i.e. the tutorial itself).

Here's the file structure:

```
<tutorial_dir>
    CMake
        <toolchain files>
    platform
        android
        ios
        linux
        macosx
        win
    src
        <tutorial .h/.c code>
```

The same structure will be used for all the tutorials, the only thing that will change will be code in the `src` subdirectory.
The platform independent OpenVG code consists essentially in four main functions:

---

```c
void tutorialInit(int surfaceWidth, int surfaceHeight);
```

It is used to create and initialize OpenVG objects (e.g. paints and paths) and default OpenVG context states (e.g. rendering quality, clear color and so on).
The drawing surface dimensions are passed as arguments.

---

```c
void tutorialDestroy(void);
```

It's called when the tutorial application is going to be closed: in this function all OpenVG resources created by the `tutorialInit` function are released.

---

```c
void tutorialResize(int surfaceWidth, int surfaceHeight);
```

It is called when the drawing surface dimensions have been changed (e.g. due to the application window resize). This function is the good place to reset those OpenVG states that dependents on the drawing surface size (e.g. scissor rectangles and alpha mask).
The new drawing surface dimensions are passed as arguments.

---

```c
void tutorialDraw(int surfaceWidth, int surfaceHeight);
```

This function is called every time the application needs to draw a new frame. As a first step the whole drawing surface is cleared with a `vgClear` instruction, then the real drawings are performed according to the tutorial topic.
The current drawing surface dimensions are passed as arguments.

---

Along with these core functions, each tutorial can implement specific actions in order to respond to user inputs (mouse/touch events).
So, the tutorial code includes the following functions too:

---

```c
void mouseLeftButtonDown(int x, int y);
```

It's called when the user pushes down the left mouse button, or when a first finger touches the device.
Usually the code will simply keep track of this state, signing it to a private variable (e.g. `mouseButton = BUTTON_LEFT`).
The push/touch location is passed as argument: coordinates are relative to the OpenVG drawing surface
(x-axis from left to right, y-axis from bottom to top, origin at the lower-left corner).

---

```c
void mouseLeftButtonUp(int x, int y);
```

It's called when the user releases the left mouse button, or the finger.
The code here will update the mouse/touch state tracked down by a previous call to the `mouseLeftButtonDown` function (e.g. `mouseButton = BUTTON_NONE`)
The release location is passed as argument: coordinates are relative to the OpenVG drawing surface
(x-axis from left to right, y-axis from bottom to top, origin at the lower-left corner).

---

```c
void mouseMove(int x, int y);
```

It's called when the user moves the mouse (or drags the finger) around the drawing surface.
The typical code will check the current mouse/touch state: if mouse button is pressed (or touch is on), the response to a "drag" action is performed (e.g. a path is moved).
The current mouse/finger location is passed as argument: coordinates are relative to the OpenVG drawing surface
(x-axis from left to right, y-axis from bottom to top, origin at the lower-left corner).

---

```c
void touchDoubleTap(int x, int y);
```

This function is specific for touch devices: it is called when the user performs the classic double-tap action.
The tap location is passed as argument: coordinates are relative to the OpenVG drawing surface
(x-axis from left to right, y-axis from bottom to top, origin at the lower-left corner).

---
