# Documentation for `modules/gapi/misc/python/package/gapi/__init__.py`

## File Metadata

- **Full Path**: `modules/gapi/misc/python/package/gapi/__init__.py`
- **File Name**: `__init__.py`
- **File Size**: 10,298 bytes
- **File Type**: .py
- **Link to Source**: [modules/gapi/misc/python/package/gapi/__init__.py](../../../../../../modules/gapi/misc/python/package/gapi/__init__.py)

## Purpose and Role

This file is located in the `modules/gapi/misc/python/package/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
__all__ = ['op', 'kernel']

import sys
import cv2 as cv

# NB: Register function in specific module
def register(mname):
    def parameterized(func):
        sys.modules[mname].__dict__[func.__name__] = func
        return func
    return parameterized


@register('cv2.gapi')
def networks(*args):
    return cv.gapi_GNetPackage(list(map(cv.detail.strip, args)))


@register('cv2.gapi')
def compile_args(*args):
    return list(map(cv.GCompileArg, args))


@register('cv2')
def GIn(*args):
    return [*args]


@register('cv2')
def GOut(*args):
    return [*args]


@register('cv2')
def gin(*args):
    return [*args]


@register('cv2.gapi')
def descr_of(*args):
    return [*args]


@register('cv2')
class GOpaque():
    # NB: Inheritance from c++ class cause segfault.
    # So just aggregate cv.GOpaqueT instead of inheritance
    def __new__(cls, argtype):
        return cv.GOpaqueT(argtype)

    class Bool():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_BOOL)

    class Int():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_INT)

    class Int64():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_INT64)

    class UInt64():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_UINT64)

    class Double():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_DOUBLE)

    class Float():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_FLOAT)

    class String():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_STRING)

    class Point():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_POINT)

    class Point2f():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_POINT2F)

    class Point3f():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_POINT3F)

    class Size():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_SIZE)

    class Rect():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_RECT)

    class Prim():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_DRAW_PRIM)

    class Any():
        def __new__(self):
            return cv.GOpaqueT(cv.gapi.CV_ANY)

@register('cv2')
class GArray():
    # NB: Inheritance from c++ class cause segfault.
    # So just aggregate cv.GArrayT instead of inheritance
    def __new__(cls, argtype):
        return cv.GArrayT(argtype)

    class Bool():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_BOOL)

    class Int():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_INT)

    class Int64():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_INT64)

    class UInt64():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_UINT64)

    class Double():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_DOUBLE)

    class Float():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_FLOAT)

    class String():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_STRING)

    class Point():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_POINT)

    class Point2f():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_POINT2F)

    class Point3f():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_POINT3F)

    class Size():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_SIZE)

    class Rect():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_RECT)

    class Scalar():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_SCALAR)

    class Mat():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_MAT)

    class GMat():
        def __new__(self):
            return cv.GArrayT(cv.gapi.CV_GMAT)

    class Prim():
        def __new__(self):
            return cv.GArray(cv.gapi.CV_DRAW_PRIM)

    class Any():
        def __new__(self):
            return cv.GArray(cv.gapi.CV_ANY)


# NB: Top lvl decorator takes arguments
def op(op_id, in_types, out_types):

    garray_types= {
            cv.GArray.Bool:    cv.gapi.CV_BOOL,
            cv.GArray.Int:     cv.gapi.CV_INT,
            cv.GArray.Int64:   cv.gapi.CV_INT64,
            cv.GArray.UInt64:  cv.gapi.CV_UINT64,
            cv.GArray.Double:  cv.gapi.CV_DOUBLE,
            cv.GArray.Float:   cv.gapi.CV_FLOAT,
            cv.GArray.String:  cv.gapi.CV_STRING,
            cv.GArray.Point:   cv.gapi.CV_POINT,
            cv.GArray.Point2f: cv.gapi.CV_POINT2F,
            cv.GArray.Point3f: cv.gapi.CV_POINT3F,
            cv.GArray.Size:    cv.gapi.CV_SIZE,
            cv.GArray.Rect:    cv.gapi.CV_RECT,
            cv.GArray.Scalar:  cv.gapi.CV_SCALAR,
            cv.GArray.Mat:     cv.gapi.CV_MAT,
            cv.GArray.GMat:    cv.gapi.CV_GMAT,
            cv.GArray.Prim:    cv.gapi.CV_DRAW_PRIM,
            cv.GArray.Any:     cv.gapi.CV_ANY
    }

    gopaque_types= {
            cv.GOpaque.Size:    cv.gapi.CV_SIZE,
            cv.GOpaque.Rect:    cv.gapi.CV_RECT,
            cv.GOpaque.Bool:    cv.gapi.CV_BOOL,
            cv.GOpaque.Int:     cv.gapi.CV_INT,
            cv.GOpaque.Int64:   cv.gapi.CV_INT64,
            cv.GOpaque.UInt64:  cv.gapi.CV_UINT64,
            cv.GOpaque.Double:  cv.gapi.CV_DOUBLE,
            cv.GOpaque.Float:   cv.gapi.CV_FLOAT,
            cv.GOpaque.String:  cv.gapi.CV_STRING,
            cv.GOpaque.Point:   cv.gapi.CV_POINT,
            cv.GOpaque.Point2f: cv.gapi.CV_POINT2F,
            cv.GOpaque.Point3f: cv.gapi.CV_POINT3F,
            cv.GOpaque.Size:    cv.gapi.CV_SIZE,
            cv.GOpaque.Rect:    cv.gapi.CV_RECT,
            cv.GOpaque.Prim:    cv.gapi.CV_DRAW_PRIM,
            cv.GOpaque.Any:     cv.gapi.CV_ANY
    }

    type2str = {
        cv.gapi.CV_BOOL:      'cv.gapi.CV_BOOL' ,
        cv.gapi.CV_INT:       'cv.gapi.CV_INT' ,
        cv.gapi.CV_INT64:     'cv.gapi.CV_INT64' ,
        cv.gapi.CV_UINT64:    'cv.gapi.CV_UINT64' ,
        cv.gapi.CV_DOUBLE:    'cv.gapi.CV_DOUBLE' ,
        cv.gapi.CV_FLOAT:     'cv.gapi.CV_FLOAT' ,
        cv.gapi.CV_STRING:    'cv.gapi.CV_STRING' ,
        cv.gapi.CV_POINT:     'cv.gapi.CV_POINT' ,
        cv.gapi.CV_POINT2F:   'cv.gapi.CV_POINT2F' ,
        cv.gapi.CV_POINT3F:   'cv.gapi.CV_POINT3F' ,
        cv.gapi.CV_SIZE:      'cv.gapi.CV_SIZE',
        cv.gapi.CV_RECT:      'cv.gapi.CV_RECT',
        cv.gapi.CV_SCALAR:    'cv.gapi.CV_SCALAR',
        cv.gapi.CV_MAT:       'cv.gapi.CV_MAT',
        cv.gapi.CV_GMAT:      'cv.gapi.CV_GMAT',
        cv.gapi.CV_DRAW_PRIM: 'cv.gapi.CV_DRAW_PRIM'
    }

    # NB: Second lvl decorator takes class to decorate
    def op_with_params(cls):
        if not in_types:
            raise Exception('{} operation should have at least one input!'.format(cls.__name__))

        if not out_types:
            raise Exception('{} operation should have at least one output!'.format(cls.__name__))

        for i, t in enumerate(out_types):
            if t not in [cv.GMat, cv.GScalar, *garray_types, *gopaque_types]:
                   raise Exception('{} unsupported output type: {} in position: {}'
                           .format(cls.__name__, t.__name__, i))

        def on(*args):
            if len(in_types) != len(args):
                raise Exception('Invalid number of input elements!\nExpected: {}, Actual: {}'
                        .format(len(in_types), len(args)))

            for i, (t, a) in enumerate(zip(in_types, args)):
                if t in garray_types:
                    if not isinstance(a, cv.GArrayT):
                        raise Exception("{} invalid type for argument {}.\nExpected: {}, Actual: {}"
                                .format(cls.__name__, i, cv.GArrayT.__name__, type(a).__name__))

                    elif a.type() != garray_types[t]:
                        raise Exception("{} invalid GArrayT type for argument {}.\nExpected: {}, Actual: {}"
                                .format(cls.__name__, i, type2str[garray_types[t]], type2str[a.type()]))

                elif t in gopaque_types:
                    if not isinstance(a, cv.GOpaqueT):
                        raise Exception("{} invalid type for argument {}.\nExpected: {}, Actual: {}"
                                .format(cls.__name__, i, cv.GOpaqueT.__name__, type(a).__name__))

                    elif a.type() != gopaque_types[t]:
                        raise Exception("{} invalid GOpaque type for argument {}.\nExpected: {}, Actual: {}"
                                .format(cls.__name__, i, type2str[gopaque_types[t]], type2str[a.type()]))

                else:
                    if t != type(a):
                        raise Exception('{} invalid input type for argument {}.\nExpected: {}, Actual: {}'
                                .format(cls.__name__, i, t.__name__, type(a).__name__))

            op = cv.gapi.__op(op_id, cls.outMeta, *args)

            out_protos = []
            for i, out_type in enumerate(out_types):
                if out_type == cv.GMat:
                    out_protos.append(op.getGMat())
                elif out_type == cv.GScalar:
                    out_protos.append(op.getGScalar())
                elif out_type in gopaque_types:
                    out_protos.append(op.getGOpaque(gopaque_types[out_type]))
                elif out_type in garray_types:
                    out_protos.append(op.getGArray(garray_types[out_type]))
                else:
                    raise Exception("""In {}: G-API operation can't produce the output with type: {} in position: {}"""
                            .format(cls.__name__, out_type.__name__, i))

            return tuple(out_protos) if len(out_protos) != 1 else out_protos[0]

        # NB: Extend operation class
        cls.id = op_id
        cls.on = staticmethod(on)
        return cls

    return op_with_params


def kernel(op_cls):
    # NB: Second lvl decorator takes class to decorate
    def kernel_with_params(cls):
        # NB: Add new members to kernel class
        cls.id      = op_cls.id
        cls.outMeta = op_cls.outMeta
        return cls

    return kernel_with_params


cv.gapi.wip.GStreamerPipeline = cv.gapi_wip_gst_GStreamerPipeline
```

## High-Level Overview

This is a Python file that may contain scripts, bindings, or utilities.

**Key Characteristics:**
- May provide Python bindings to C++ code
- Could be a utility script for build/test automation
- Might implement examples or tutorials
- Uses Python idioms and standard library


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **cls**: A class/struct defined in this file
- **Double**: A class/struct defined in this file
- **Point3f**: A class/struct defined in this file
- **Rect**: A class/struct defined in this file
- **Any**: A class/struct defined in this file
- **Mat**: A class/struct defined in this file
- **Point**: A class/struct defined in this file
- **GMat**: A class/struct defined in this file
- **UInt64**: A class/struct defined in this file
- **Int64**: A class/struct defined in this file
- **GArray**: A class/struct defined in this file
- **GOpaque**: A class/struct defined in this file
- **Scalar**: A class/struct defined in this file
- **Int**: A class/struct defined in this file
- **Size**: A class/struct defined in this file
- **Prim**: A class/struct defined in this file
- **Bool**: A class/struct defined in this file
- **to**: A class/struct defined in this file
- **Float**: A class/struct defined in this file
- **cause**: A class/struct defined in this file
- **String**: A class/struct defined in this file
- **Point2f**: A class/struct defined in this file

### Functions and Methods

- **kernel()**: A function/method defined in this file
- **in()**: A function/method defined in this file
- **networks()**: A function/method defined in this file
- **kernel_with_params()**: A function/method defined in this file
- **register()**: A function/method defined in this file
- **__new__()**: A function/method defined in this file
- **GIn()**: A function/method defined in this file
- **op_with_params()**: A function/method defined in this file
- **on()**: A function/method defined in this file
- **parameterized()**: A function/method defined in this file
- **GOut()**: A function/method defined in this file
- **return()**: A function/method defined in this file
- **op()**: A function/method defined in this file
- **compile_args()**: A function/method defined in this file
- **descr_of()**: A function/method defined in this file
- **gin()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `c`
- `sys`
- `cv2`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

