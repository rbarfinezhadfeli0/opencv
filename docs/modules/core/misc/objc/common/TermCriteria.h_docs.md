# Documentation for `modules/core/misc/objc/common/TermCriteria.h`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/TermCriteria.h`
- **File Name**: `TermCriteria.h`
- **File Size**: 1,538 bytes
- **File Type**: .h
- **Link to Source**: [modules/core/misc/objc/common/TermCriteria.h](../../../../../modules/core/misc/objc/common/TermCriteria.h)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  TermCriteria.h
//
//  Created by Giles Payne on 2019/10/08.
//

#pragma once

#ifdef __cplusplus
#import "opencv2/core.hpp"
#else
#define CV_EXPORTS
#endif

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

/**
* Class representing termination criteria for iterative algorithms.
*/
CV_EXPORTS @interface TermCriteria : NSObject

#pragma mark - Properties

@property(class, readonly) int COUNT;
@property(class, readonly) int EPS;
@property(class, readonly) int MAX_ITER;

@property int type;
@property int maxCount;
@property double epsilon;
#ifdef __cplusplus
@property(readonly) cv::TermCriteria& nativeRef;
#endif

#pragma mark - Constructors

- (instancetype)init;
- (instancetype)initWithType:(int)type maxCount:(int)maxCount epsilon:(double)epsilon;
- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals;
#ifdef __cplusplus
+ (instancetype)fromNative:(cv::TermCriteria&)nativeTermCriteria;
#endif

#pragma mark - Methods

/**
* Set the termination criteria values from the values of an array
* @param vals The array of values from which to set the termination criteria values
*/
- (void)set:(NSArray<NSNumber*>*)vals NS_SWIFT_NAME(set(vals:));

#pragma mark - Common Methods

/**
* Clone object
*/
- (TermCriteria*)clone;

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

- **TermCriteria**: A class/struct defined in this file

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

