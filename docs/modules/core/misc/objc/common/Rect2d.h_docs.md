# Documentation for `modules/core/misc/objc/common/Rect2d.h`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Rect2d.h`
- **File Name**: `Rect2d.h`
- **File Size**: 2,067 bytes
- **File Type**: .h
- **Link to Source**: [modules/core/misc/objc/common/Rect2d.h](../../../../../modules/core/misc/objc/common/Rect2d.h)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Rect.h
//
//  Created by Giles Payne on 2019/10/09.
//

#pragma once

#ifdef __cplusplus
#import "opencv2/core.hpp"
#else
#define CV_EXPORTS
#endif

@class Point2d;
@class Size2d;

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

/**
* Represents a rectange the coordinate and dimension values of which are of type `double`
*/
CV_EXPORTS @interface Rect2d : NSObject

#pragma mark - Properties

@property double x;
@property double y;
@property double width;
@property double height;
#ifdef __cplusplus
@property(readonly) cv::Rect2d& nativeRef;
#endif

#pragma mark - Constructors

- (instancetype)init;
- (instancetype)initWithX:(double)x y:(double)y width:(double)width height:(double)height;
- (instancetype)initWithPoint:(Point2d*)point1 point:(Point2d*)point2;
- (instancetype)initWithPoint:(Point2d*)point size:(Size2d*)size;
- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals;

#ifdef __cplusplus
+ (instancetype)fromNative:(cv::Rect2d&)point;
#endif

#pragma mark - Methods

/**
* Returns the top left coordinate of the rectangle
*/
- (Point2d*)tl;

/**
* Returns the bottom right coordinate of the rectangle
*/
- (Point2d*)br;

/**
* Returns the size of the rectangle
*/
- (Size2d*)size;

/**
* Returns the area of the rectangle
*/
- (double)area;

/**
* Determines if the rectangle is empty
*/
- (BOOL)empty;

/**
* Determines if the rectangle contains a given point
* @param point The point
*/
- (BOOL)contains:(Point2d*)point;

/**
* Set the rectangle coordinates and dimensions from the values of an array
* @param vals The array of values from which to set the rectangle coordinates and dimensions
*/
- (void)set:(NSArray<NSNumber*>*)vals NS_SWIFT_NAME(set(vals:));

#pragma mark - Common Methods

/**
* Clone object
*/
- (Rect2d*)clone;

/**
* Compare for equality
* @param other Object to compare
*/
- (BOOL)isEqual:(nullable id)object;

/**
* Calculate hash value for this object
*/
- (NSUInteger)hash;

/**
* Returns a string that describes the contents of the object
*/
- (NSString*)description;

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

- **Point2d**: A class/struct defined in this file
- **Size2d**: A class/struct defined in this file
- **Rect2d**: A class/struct defined in this file

### Functions and Methods

- **__cplusplus()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `the`
- `which`


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

