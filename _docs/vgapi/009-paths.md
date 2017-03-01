---
layout: default
title: "Paths"
date: 2017-01-01 08:00:00 +0100
chapter: 9
categories: [vgapi]
---

# Paths

Paths are the heart of the OpenVG API. All geometry to be drawn must be defined in terms of one or more paths. Paths are defined by a sequence of segment commands. Each segment command in the standard format may specify a move, a straight line segment, a quadratic or cubic Bézier segment, or an elliptical arc.

### Segment Commands [8.5.2]

Reference points are defined as (all are initially `(0, 0)`): 

 * `(sx, sy)`: beginning of the current subpath;  
 * `(ox, oy)`: last point of the previous segment;  
 * `(px, py)`: last internal control point of the previous segment if the segment was a (regular or smooth) quadratic or cubic Bézier, or else the last point of the previous segment.  

The following table describes each segment command type and the side effects of the segment command on the termination of the current subpath.

| `VGPathSegment` | Coordinates | Implicit Points | Description (Side Effects) |
| --------------- | ----------- | --------------- | -------------------------- |
| `VG_CLOSE_PATH` | none | | (px, py) = (ox, oy) = (sx, sy)<br> End current subpath. | 
| `VG_MOVE_TO` | x0, y0 | | (sx, sy) = (px, py) = (ox, oy) = (x0, y0)<br> End current subpath. |
| `VG_LINE_TO` | x0, y0 | | (px, py) = (ox, oy) = (x0, y0) |
| `VG_HLINE_TO` | x0 | y0 = oy | (px, py) = (x0, oy), ox = x0 |
| `VG_VLINE_TO` | y0 | x0 = ox | (px, py) = (ox, y0), oy = y0 |
| `VG_QUAD_TO` | x0, y0, x1, y1 | | (px, py) = (x0, y0)<br> (ox, oy) = (x1, y1) |
| `VG_CUBIC_TO` | x0, y0, x1, y1, x2, y2 | | (px, py) = (x1, y1)<br> (ox, oy) = (x2, y2) |
| `VG_SQUAD_TO` | x1, y1 | (x0, y0) = (2 * ox ‐ px, 2 * oy ‐ py) | (px, py) = (2 * ox - px, 2 * oy - py)<br> (ox, oy) = (x1, y1) |
| `VG_SCUBIC_TO` | x1, y1, x2, y2 | (x0, y0) = (2 * ox ‐ px, 2 * oy ‐ py) | (px, py) = (x1, y1)<br> (ox, oy) = (x2, y2) |
| `VG_SCCWARC_TO` | rh, rv, rot, x0, y0 | | (px, py) = (ox, oy) = (x0, y0) |
| `VG_SCWARC_TO` | rh, rv, rot, x0, y0 | | (px, py) = (ox, oy) = (x0, y0) |
| `VG_LCCWARC_TO` | rh, rv, rot, x0, y0 | | (px, py) = (ox, oy) = (x0, y0) |
| `VG_LCWARC_TO` | rh, rv, rot, x0, y0 | | (px, py) = (ox, oy) = (x0, y0) |
{:.rwd-table}

---

## Path Operations [8.6]

### Create and Destroy Path [8.6.2]

```c
VGPath vgCreatePath(VGint pathFormat,
                    VGPathDatatype datatype,
                    VGfloat scale,
                    VGfloat bias,
                    VGint segCapacityHint,
                    VGint coordCapacityHint,
                    VGbitfield capabilities)
```
Create a new path that is ready to accept segment data and returns a `VGPath` handle to it.  
The path data will be formatted in the format given by `pathFormat`, typically `VG_PATH_FORMAT_STANDARD`.  
The `datatype` parameter contains a value from the `VGPathDatatype` enumeration indicating the datatype that will be used for coordinate data.  
The `capabilities` argument is a bitwise OR of the desired `VGPathCapabilities` values. If an error occurs, `VG_INVALID_HANDLE` is returned.  
The scale and bias parameters are used to interpret each coordinate of the path data: an incoming coordinate value v will be interpreted as the value (`scale` * v + `bias`). The parameter `scale` must not equal 0.

---
{:.hrlight}

```c
void vgClearPath(VGPath path, VGbitfield capabilities)
```
Remove all segment command and coordinate data associated with the given `path`.  
The handle continues to be valid for use.

---
{:.hrlight}

```c
void vgDestroyPath(VGPath path)
```
Deallocate a `path`, releasing any resources associated with it.  
The handle becomes invalid in all contexts that shared it.

---

#### Query & Modify Path Capabilities [8.6.4]

```c
VGbitfield vgGetPathCapabilities(VGPath path)
```
Query the set of capabilities for a path as a bitwise OR of `VGPathCapabilities` constants.  
If an error occurs, 0 is returned.

---
{:.hrlight}

```c
void vgRemovePathCapabilities(VGPath path, VGbitfield capabilities)
```
Reduce the set of capabilities for a path.  
The `capabilities` argument is a bitwise OR of the `VGPathCapabilities` values whose removal is requested.

---

### Copy Data Between Paths [8.6.5-6]
  
```c
void vgAppendPath(VGPath dstPath, VGPath srcPath)
```
Append a copy of all path segments from `srcPath` onto the end of the existing data in `dstPath`.  
The `VG_PATH_CAPABILITY_APPEND_FROM` capability must be enabled for `srcPath`, and the `VG_PATH_CAPABILITY_APPEND_TO` capability must be enabled for `dstPath`.

---
{:.hrlight}

```c
void vgAppendPathData(VGPath dstPath,
                      VGint numSeg,
                      const VGubyte* pathSeg,
                      const void* pathData)
```
Append data taken from `pathData` to the given path `dstPath`.  
The data are formatted using the path format of `dstPath`.  
The `pathData` pointer must be aligned on a 1, 2, or 4-byte boundary depending on the size of the coordinate datatype.  
The `VG_PATH_CAPABILITY_APPEND_TO` capability must be enabled for path.  
Each incoming coordinate value, regardless of datatype, is transformed by the scale factor and bias of the path.

---

### Modify Path Data [8.6.7]

```c
void vgModifyPathCoords(VGPath dstPath,
                        VGint startIdx,
                        VGint sumSeg,
                        const void* pathData)
```
Modify the coordinate data for a contiguous range of segments of `dstPath`, starting at `startIndex` (where 0 is the index of the first path segment) and having length `numSegments`.  
The `pathData` pointer must be aligned on a 1, 2, or 4-byte boundary depending on the size of the coordinate datatype.  
The `VG_PATH_CAPABILITY_MODIFY` capability must be enabled for path.  
Each incoming coordinate value, regardless of datatype, is transformed by the scale factor and bias of the path.

---

### Transform Path [8.6.8]

```c
void vgTransformPath(VGPath dstPath, VGPath srcPath)
```
Append a transformed copy of `srcPath` to the current contents of `dstPath`.  
The appended path is equivalent to the results of applying the current pathuser-to-surface transformation (`VG_MATRIX_PATH_USER_TO_SURFACE`) to `srcPath`.  
The `VG_PATH_CAPABILITY_TRANSFORM_FROM` capability must be enabled for `srcPath`, and the `VG_PATH_CAPABILITY_TRANSFORM_TO` capability must be enabled for `dstPath`.

---

### Interpolate Between Paths [8.6.9]

```c
VGboolean vgInterpolatePath(VGPath dstPath,
                            VGPath startPath,
                            VGPath endPath,
                            VGfloat amount)
```
Append a path, defined by interpolation (or extrapolation) between the paths `startPath` and `endPath` by the given amount, to the path `dstPath`. It returns `VG_TRUE` if interpolation was successful, `VG_FALSE` otherwise.  
If interpolation is unsuccessful, `dstPath` is left unchanged.  
The `VG_PATH_CAPABILITY_INTERPOLATE_FROM` capability must be enabled for both of `startPath` and `endPath`, and the `INTERPOLATE_TO` capability must be enabled for `dstPath`.

---

### Length of Path [8.6.10]

```c
VGfloat vgPathLength(VGPath path, VGint startSeg, VGint numSeg)
```
Return the length of a given portion of a path in the user coordinate system (that is, in the path’s own coordinate system, disregarding any matrix settings).  
Only the subpath consisting of the `numSegments` path segments beginning with `startSegment` (where the initial path segment has index 0) is used. If an error occurs, -1.0f is returned.  
The `VG_PATH_CAPABILITY_PATH_LENGTH` capability must be enabled for path.

---

### Position & Tangent Along Path [8.6.11]

```c
void vgPointAlongPath(VGPath path,
                      VGint startSeg,
                      VGint numSeg,
                      VGfloat distance,
                      VGfloat* x,
                      VGfloat* y,
                      VGfloat* tanX,
                      VGfloat* tanY)
```
Return the point lying a given distance along a given portion of a path and the unit-length tangent vector at that point.  
Only the subpath consisting of the `numSegments` path segments beginning with `startSegment` (where the initial path segment has index 0) is used.  
If `distance` is less than or equal to 0, the starting point of the path is used. If `distance` is greater than or equal to the path length, the visual ending point of the path is used.  
The `VG_PATH_CAPABILITY_POINT_ALONG_PATH` capability must be enabled for path.

---

### Query Bounding Box [8.6.12]

```c
void vgPathBounds(VGPath path,
                  VGfloat* minx,
                  VGfloat* miny,
                  VGfloat* width,
                  VGfloat* height)
```
Returns an axis-aligned bounding box that tightly bounds the interior of the given `path`.  
Stroking parameters are ignored.  
The `VG_PATH_CAPABILITY_PATH_BOUNDS` capability must be enabled for `path`.

---
{:.hrlight}

```c
void vgPathTransformedBounds(VGPath path,
                             VGfloat* minx,
                             VGfloat* miny,
                             VGfloat* width,
                             VGfloat* height)
```
Return an axis-aligned bounding box that is guaranteed to enclose the geometry of the given path following transformation by the current `VG_MATRIX_PATH_USER_TO_SURFACE` transform.  
The returned bounding box is not guaranteed to fit tightly around the path geometry.  
The `VG_PATH_CAPABILITY_PATH_TRANSFORMED_BOUNDS` capability must be enabled for `path`.

---

## Draw Path [8.8]

```c
VGfloat vgDrawPath(VGPath path, VGbitfield paintModes)
```
Filling and stroking are performed by the `vgDrawPath` function.  
The `paintModes` argument is a bitwise OR of values from the `VGPaintMode` enumeration, determining whether the path is to be filled (`VG_FILL_PATH`), stroked (`VG_STROKE_PATH`), or both (`VG_FILL_PATH | VG_STROKE_PATH`).  
If both filling and stroking are to be performed, the path is first filled, then stroked.

---

### Path Object Parameter [8.6.3]

Values from the `VGPathParamType` enumeration may be used as the `paramType` argument to `vgGetParameter` to query various features of a path.  
All of the parameters defined by `VGPathParamType` are read-only.

| Parameter name | Parameter type | Notes |
| -------------- | -------------- | ----- |
| `VG_PATH_FORMAT` | `VGint` | Fixed and equal to `VG_PATH_FORMAT_STANDARD (0)` |
| `VG_PATH_DATATYPE` | `VGPathDatatype` | One of the following: `VG_PATH_DATATYPE_S_{8, 16, 32}, VG_PATH_DATATYPE_F` |
| `VG_PATH_BIAS` | `VGfloat` | Specified through the `vgCreatePath` function |
| `VG_PATH_SCALE` | `VGfloat` | Specified through the `vgCreatePath` function |
| `VG_PATH_NUM_SEGMENTS` | `VGint` ||
| `VG_PATH_NUM_COORDS` | `VGint` ||
{:.rwd-table}

---

