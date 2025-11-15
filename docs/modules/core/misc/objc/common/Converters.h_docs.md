# Documentation for `modules/core/misc/objc/common/Converters.h`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Converters.h`
- **File Name**: `Converters.h`
- **File Size**: 3,610 bytes
- **File Type**: .h
- **Link to Source**: [modules/core/misc/objc/common/Converters.h](../../../../../modules/core/misc/objc/common/Converters.h)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Converters.h
//
//  Created by Giles Payne on 2020/03/03.
//

#pragma once

#ifdef __cplusplus
#import <opencv2/core.hpp>
#else
#define CV_EXPORTS
#endif

#import <Foundation/Foundation.h>
#import "Mat.h"
#import "CvType.h"
#import "Point2i.h"
#import "Point2f.h"
#import "Point2d.h"
#import "Point3i.h"
#import "Point3f.h"
#import "Point3d.h"
#import "Rect2i.h"
#import "Rect2d.h"
#import "KeyPoint.h"
#import "DMatch.h"
#import "RotatedRect.h"

NS_ASSUME_NONNULL_BEGIN

CV_EXPORTS @interface Converters : NSObject

+ (Mat*)vector_Point_to_Mat:(NSArray<Point2i*>*)pts NS_SWIFT_NAME(vector_Point_to_Mat(_:));

+ (NSArray<Point2i*>*)Mat_to_vector_Point:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Point(_:));

+ (Mat*)vector_Point2f_to_Mat:(NSArray<Point2f*>*)pts NS_SWIFT_NAME(vector_Point2f_to_Mat(_:));

+ (NSArray<Point2f*>*)Mat_to_vector_Point2f:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Point2f(_:));

+ (Mat*)vector_Point2d_to_Mat:(NSArray<Point2d*>*)pts NS_SWIFT_NAME(vector_Point2d_to_Mat(_:));

+ (NSArray<Point2d*>*)Mat_to_vector_Point2d:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Point2d(_:));

+ (Mat*)vector_Point3i_to_Mat:(NSArray<Point3i*>*)pts NS_SWIFT_NAME(vector_Point3i_to_Mat(_:));

+ (NSArray<Point3i*>*)Mat_to_vector_Point3i:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Point3i(_:));

+ (Mat*)vector_Point3f_to_Mat:(NSArray<Point3f*>*)pts NS_SWIFT_NAME(vector_Point3f_to_Mat(_:));

+ (NSArray<Point3f*>*)Mat_to_vector_Point3f:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Point3f(_:));

+ (Mat*)vector_Point3d_to_Mat:(NSArray<Point3d*>*)pts NS_SWIFT_NAME(vector_Point3d_to_Mat(_:));

+ (NSArray<Point3d*>*)Mat_to_vector_Point3d:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Point3d(_:));

+ (Mat*)vector_float_to_Mat:(NSArray<NSNumber*>*)fs NS_SWIFT_NAME(vector_float_to_Mat(_:));

+ (NSArray<NSNumber*>*)Mat_to_vector_float:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_float(_:));

+ (Mat*)vector_uchar_to_Mat:(NSArray<NSNumber*>*)us NS_SWIFT_NAME(vector_uchar_to_Mat(_:));

+ (NSArray<NSNumber*>*)Mat_to_vector_uchar:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_uchar(_:));

+ (Mat*)vector_char_to_Mat:(NSArray<NSNumber*>*)cs NS_SWIFT_NAME(vector_char_to_Mat(_:));

+ (NSArray<NSNumber*>*)Mat_to_vector_char:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_char(_:));

+ (Mat*)vector_int_to_Mat:(NSArray<NSNumber*>*)is NS_SWIFT_NAME(vector_int_to_Mat(_:));

+ (NSArray<NSNumber*>*)Mat_to_vector_int:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_int(_:));

+ (Mat*)vector_Rect_to_Mat:(NSArray<Rect2i*>*)rs NS_SWIFT_NAME(vector_Rect_to_Mat(_:));

+ (NSArray<Rect2i*>*)Mat_to_vector_Rect:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Rect(_:));

+ (Mat*)vector_Rect2d_to_Mat:(NSArray<Rect2d*>*)rs NS_SWIFT_NAME(vector_Rect2d_to_Mat(_:));

+ (NSArray<Rect2d*>*)Mat_to_vector_Rect2d:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_Rect2d(_:));

+ (Mat*)vector_KeyPoint_to_Mat:(NSArray<KeyPoint*>*)kps NS_SWIFT_NAME(vector_KeyPoint_to_Mat(_:));

+ (NSArray<KeyPoint*>*)Mat_to_vector_KeyPoint:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_KeyPoint(_:));

+ (Mat*)vector_double_to_Mat:(NSArray<NSNumber*>*)ds NS_SWIFT_NAME(vector_double_to_Mat(_:));

+ (NSArray<NSNumber*>*)Mat_to_vector_double:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_double(_:));

+ (Mat*)vector_DMatch_to_Mat:(NSArray<DMatch*>*)matches NS_SWIFT_NAME(vector_DMatch_to_Mat(_:));

+ (NSArray<DMatch*>*)Mat_to_vector_DMatch:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_DMatch(_:));

+ (Mat*)vector_RotatedRect_to_Mat:(NSArray<RotatedRect*>*)rs NS_SWIFT_NAME(vector_RotatedRect_to_Mat(_:));

+ (NSArray<RotatedRect*>*)Mat_to_vector_RotatedRect:(Mat*)mat NS_SWIFT_NAME(Mat_to_vector_RotatedRect(_:));

@end

NS_ASSUME_NONNULL_END
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **Converters**: A class/struct defined in this file

### Functions and Methods

- **__cplusplus()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

