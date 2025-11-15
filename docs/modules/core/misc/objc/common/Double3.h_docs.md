# Documentation for `modules/core/misc/objc/common/Double3.h`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Double3.h`
- **File Name**: `Double3.h`
- **File Size**: 1,492 bytes
- **File Type**: .h
- **Link to Source**: [modules/core/misc/objc/common/Double3.h](../../../../../modules/core/misc/objc/common/Double3.h)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Double3.h
//
//  Created by Giles Payne on 2020/05/22.
//

#pragma once

#ifdef __cplusplus
#import "opencv2/core.hpp"
#else
#define CV_EXPORTS
#endif

#import <Foundation/Foundation.h>

@class Mat;

NS_ASSUME_NONNULL_BEGIN

/**
* Simple wrapper for a vector of three `double`
*/
CV_EXPORTS @interface Double3 : NSObject

#pragma mark - Properties

/**
* First vector element
*/
@property double v0;

/**
* Second vector element
*/
@property double v1;

/**
* Third vector element
*/
@property double v2;


#ifdef __cplusplus
/**
* The wrapped vector
*/
@property(readonly) cv::Vec3d& nativeRef;
#endif

#pragma mark - Constructors

/**
* Create zero-initialize vecior
*/
-(instancetype)init;

/**
* Create vector with specified element values
* @param v0 First element
* @param v1 Second element
* @param v2 Third element
*/
-(instancetype)initWithV0:(double)v0 v1:(double)v1 v2:(double)v2;

/**
* Create vector with specified element values
* @param vals array of element values
*/
-(instancetype)initWithVals:(NSArray<NSNumber*>*)vals;
#ifdef __cplusplus
+(instancetype)fromNative:(cv::Vec3d&)vec3d;
#endif

/**
* Update vector with specified element values
* @param vals array of element values
*/
-(void)set:(NSArray<NSNumber*>*)vals NS_SWIFT_NAME(set(vals:));

/**
* Get vector as an array
*/
-(NSArray<NSNumber*>*)get;

#pragma mark - Common Methods

/**
* Compare for equality
* @param other Object to compare
*/
-(BOOL)isEqual:(nullable id)other;

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

- **Mat**: A class/struct defined in this file
- **Double3**: A class/struct defined in this file

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

