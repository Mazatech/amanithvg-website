---
layout: default
title: "Multi thread environment"
date: 2018-11-01 08:00:00 +0100
chapter: 6
categories: [desc]
headline: "How to use AmanithVG in a multi thread application"
image: "amanithvg-logo.png"
keywords: "amanithvg thread concurrency parallelism"
---

# How to use AmanithVG in a multi-thread application/environment

AmanithVG is thread-safe: all the exposed functions can be called from multiple threads at the same time. When using AmanithVG in a multi-thread application, you have to take some important aspects into consideration:

- according to OpenVG specifications, `VGHandle` type is a 32bit integer and it is used to identify OpenVG objects (paths, paints, images, mask layers, fonts)
- AmanithVG uses 5 bits of `VGHandle` type to identify the thread that created the OpenVG object, so that AmanithVG can handle a maximum of 32 (equal to 2^5) current threads, with a maximum of 134,217,728 (equal to 2^27) instantiable objects (`VGPath`, `VGPaint`, `VGImage`, `VGMaskLayer`, `VGFont`) for each thread
- the actual maximum number of current threads that AmanithVG handles can be also queried at runtime by calling `vgConfigGetMZT(VG_CONFIG_MAX_CURRENT_THREADS_MZT)`
- the `vgTerminateMZT` function should be called only after all threads have completed their execution

Here is a general example of a possible multi-thread application that uses AmanithVG to draw vector graphic contents concurrently:

```c
// global mutex
Mutex mutex;
// global list of OpenVG contents to be rendered by threads
ContentsList contents;
// index of the next OpenVG content to be rendered
int current;

static void threadRender(Content c,
                         void* vgContext) {

    // create an OpenVG drawing surface; in general, surface
    // dimensions are dictated by the content itself
    void* vgSurface = vgPrivSurfaceCreateMZT(c.width, c.height,
                                             // non-linear premultiplied color space
                                             VG_FALSE, VG_TRUE,
                                             // alpha mask enabled
                                             VG_TRUE);
    // get the actual dimensions
    int width = vgPrivGetSurfaceWidthMZT(vgSurface);
    int height = vgPrivGetSurfaceHeightMZT(vgSurface);

    // bind context and surface
    vgPrivMakeCurrentMZT(vgContext, vgSurface);

    // clear the drawing surface
    vgClear(0, 0, width, height);

    // OpenVG rendering code, according to content
    openvgContentRender(c);

#ifdef AMANITHVG_GLE
    // AmanithVG GLE
    //
    // use the native OpenGL (ES) swap buffers function, or do
    // something with the bound OpenGL framebuffer object
#else
    // AmanithVG SRE
    //
    // get surface pixels by calling vgPrivGetSurfacePixelsMZT(vgSurface)
    // and do something with them (e.g. dump a PNG, send the content
    // via network, blit the pixels on screen, ...)
#endif

    // acknowledge AmanithVG that we have performed a swapbuffers-like operation
    vgPostSwapBuffersMZT();

    // unbind context and surface
    vgPrivMakeCurrentMZT(NULL, NULL);

    // destroy the OpenVG drawing surface
    vgPrivSurfaceDestroyMZT(vgSurface);
}

// thread function
static void threadFunc(void) {

    bool done = false;
    // create and initialize an OpenVG context
    void* vgContext = vgPrivContextCreateMZT(NULL);

    while (!done) {
        // lock the global mutex
        mutex.lock();
        // if there are still OpenVG contents to render...
        if (current < contents.count) {
            // ... extract the next one
            Content c = contents[current++];
            // unlock the global mutex
            mutex.unlock();
            // draw it
            threadRender(c, vgContext);
        }
        else {
            // unlock the global mutex
            mutex.unlock();
            // there are no more SVG to render, this thread can exit
            done = true;
        }
    }

    // destroy the OpenVG context
    vgPrivContextDestroyMZT(vgContext);
}

// main entry point
int main(int argc,
         char *argv[]) {

    int i;
    ThreadList threads;
    // the maximum number of different threads that can "work"
    // (e.g. create surfaces, draw paths, etc) concurrently
    // in AmanithVG
    int vgThreads = (int)vgConfigGetMZT(VG_CONFIG_MAX_CURRENT_THREADS_MZT);
    // get the number of concurrent threads supported by the
    // hardware/os platform
    int hardwareThreads = hardware_concurrency();
    // the actual number of threads that we'll spawn
    int maxThreads = min(vgThreads, hardwareThreads);

    // initialize AmanithVG library, after initialization it is possible
    // to create contexts and drawing surfaces; in a multi-thread program
    // it is recommended to initialize the library once before the creation
    // of threads
    vgInitializeMZT();

    // set quality parameters
    vgConfigSetMZT(VG_CONFIG_CURVES_QUALITY_MZT, 75.0f);
    vgConfigSetMZT(VG_CONFIG_RADIAL_GRADIENTS_QUALITY_MZT, 75.0f);
    vgConfigSetMZT(VG_CONFIG_CONICAL_GRADIENTS_QUALITY_MZT, 75.0f);
    // set other parameters, if desired...

    // spwan rendering threads
    for (i = 0; i < maxThreads; i++) {
        // start a new thread
        threads.push(threadFunc);
    }

    // wait for all threads to finish
    for (i = 0; i < threads.count; i++) {
        threads[i].join();
    }

    // release AmantihVG library
    vgTerminateMZT();

    return EXIT_SUCCESS;
}
```

---
