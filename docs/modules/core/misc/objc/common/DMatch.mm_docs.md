# Documentation for `modules/core/misc/objc/common/DMatch.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/DMatch.mm`
- **File Name**: `DMatch.mm`
- **File Size**: 2,567 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/DMatch.mm](../../../../../modules/core/misc/objc/common/DMatch.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  DMatch.m
//
//  Created by Giles Payne on 2019/12/25.
//

#import "DMatch.h"

@implementation DMatch {
    cv::DMatch native;
}

- (int)queryIdx {
    return native.queryIdx;
}

- (void)setQueryIdx:(int)queryIdx {
    native.queryIdx = queryIdx;
}

- (int)trainIdx {
    return native.trainIdx;
}

- (void)setTrainIdx:(int)trainIdx {
    native.trainIdx = trainIdx;
}

- (int)imgIdx {
    return native.imgIdx;
}

- (void)setImgIdx:(int)imgIdx {
    native.imgIdx = imgIdx;
}

- (float)distance {
    return native.distance;
}

- (void)setDistance:(float)distance {
    native.distance = distance;
}

- (cv::DMatch&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithQueryIdx:-1 trainIdx:-1 distance:FLT_MAX];
}

- (instancetype)initWithQueryIdx:(int)queryIdx trainIdx:(int)trainIdx distance:(float)distance {
    return [self initWithQueryIdx:queryIdx trainIdx:trainIdx imgIdx:-1 distance:distance];
}

- (instancetype)initWithQueryIdx:(int)queryIdx trainIdx:(int)trainIdx imgIdx:(int)imgIdx distance:(float)distance {
    self = [super init];
    if (self != nil) {
        self.queryIdx = queryIdx;
        self.trainIdx = trainIdx;
        self.imgIdx = imgIdx;
        self.distance = distance;
    }
    return self;
}

+ (instancetype)fromNative:(cv::DMatch&)dMatch {
    return [[DMatch alloc] initWithQueryIdx:dMatch.queryIdx trainIdx:dMatch.trainIdx imgIdx:dMatch.imgIdx distance:dMatch.distance];
}

- (BOOL)lessThan:(DMatch*)it {
    return self.distance < it.distance;
}


- (DMatch*)clone {
    return [[DMatch alloc] initWithQueryIdx:self.queryIdx trainIdx:self.trainIdx imgIdx:self.imgIdx distance:self.distance];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[DMatch class]]) {
        return NO;
    } else {
        DMatch* dMatch = (DMatch*)other;
        return self.queryIdx == dMatch.queryIdx && self.trainIdx == dMatch.trainIdx && self.imgIdx == dMatch.imgIdx && self.distance == dMatch.distance;
    }
}

#define FLOAT_TO_BITS(x)  ((Cv32suf){ .f = x }).i

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + self.queryIdx;
    result = prime * result + self.trainIdx;
    result = prime * result + self.imgIdx;
    result = prime * result + FLOAT_TO_BITS(self.distance);
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"DMatch { queryIdx: %d, trainIdx: %d, imgIdx: %d, distance: %f}", self.queryIdx, self.trainIdx, self.imgIdx, self.distance];
}

@end
```

## High-Level Overview

This is a .mm source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.


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

