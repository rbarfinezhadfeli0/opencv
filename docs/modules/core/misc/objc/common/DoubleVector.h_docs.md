# Documentation for `modules/core/misc/objc/common/DoubleVector.h`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/DoubleVector.h`
- **File Name**: `DoubleVector.h`
- **File Size**: 1,761 bytes
- **File Type**: .h
- **Link to Source**: [modules/core/misc/objc/common/DoubleVector.h](../../../../../modules/core/misc/objc/common/DoubleVector.h)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  DoubleVector.h
//
//  Created by Giles Payne on 2020/01/04.
//

#pragma once

#import <Foundation/Foundation.h>
#ifdef __cplusplus
#import <vector>
#endif
#import "CVObjcUtil.h"

NS_ASSUME_NONNULL_BEGIN

/**
* Utility class to wrap a `std::vector<double>`
*/
CV_EXPORTS @interface DoubleVector : NSObject

#pragma mark - Constructors

/**
* Create DoubleVector and initialize with the  contents of an NSData object
* @param data  NSData containing raw double array
*/
-(instancetype)initWithData:(NSData*)data;

/**
* Create DoubleVector and initialize with the  contents of another DoubleVector object
* @param src  DoubleVector containing data to copy
*/
-(instancetype)initWithVector:(DoubleVector*)src;

#ifdef __OBJC__
/**
* Create DoubleVector from raw C array
* @param array The raw C array
* @elements elements The number of elements in the array
*/
-(instancetype)initWithNativeArray:(double*)array elements:(int)elements;
#endif

#ifdef __cplusplus
/**
* Create DoubleVector from std::vector<double>
* @param src The std::vector<double> object to wrap
*/
-(instancetype)initWithStdVector:(std::vector<double>&)src;
+(instancetype)fromNative:(std::vector<double>&)src;
#endif

#pragma mark - Properties

/**
* Length of the vector
*/
@property(readonly) size_t length;

#ifdef __OBJC__
/**
* Raw C array
*/
@property(readonly) double* nativeArray;
#endif

#ifdef __cplusplus
/**
* The wrapped std::vector<double> object
*/
@property(readonly) std::vector<double>& nativeRef;
#endif

/**
* NSData object containing the raw double data
*/
@property(readonly) NSData* data;

#pragma mark - Accessor method

/**
* Return array element
* @param index Index of the array element to return
*/
-(double)get:(NSInteger)index;

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

- **to**: A class/struct defined in this file
- **DoubleVector**: A class/struct defined in this file

### Functions and Methods

- **__OBJC__()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `std`
- `raw`


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

