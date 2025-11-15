# Documentation for `modules/core/misc/objc/common/Point2d.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Point2d.mm`
- **File Name**: `Point2d.mm`
- **File Size**: 2,088 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Point2d.mm](../../../../../modules/core/misc/objc/common/Point2d.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Point2d.m
//
//  Created by Giles Payne on 2019/10/09.
//

#import "Point2d.h"
#import "Rect2d.h"

@implementation Point2d {
    cv::Point2d native;
}

- (double)x {
    return native.x;
}

- (void)setX:(double)val {
    native.x = val;
}

- (double)y {
    return native.y;
}

- (void)setY:(double)val {
    native.y = val;
}

- (cv::Point2d&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithX:0 y:0];
}

- (instancetype)initWithX:(double)x y:(double)y {
    self = [super init];
    if (self) {
        self.x = x;
        self.y = y;
    }
    return self;
}

- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [super init];
    if (self) {
        [self set:vals];
    }
    return self;
}

+ (instancetype)fromNative:(cv::Point2d&)point {
    return [[Point2d alloc] initWithX:point.x y:point.y];
}

- (void)update:(cv::Point2d&)point {
    self.x = point.x;
    self.y = point.y;
}

- (Point2d*) clone {
    return [[Point2d alloc] initWithX:self.x y:self.y];
}

- (double)dot:(Point2d*)point {
    return self.x * point.x + self.y * point.y;
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.x = (vals != nil && vals.count > 0) ? vals[0].doubleValue : 0;
    self.y = (vals != nil && vals.count > 1) ? vals[1].doubleValue : 0;
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Point2d class]]) {
        return NO;
    } else {
        Point2d* point = (Point2d*)other;
        return self.x == point.x && self.y == point.y;
    }
}

- (BOOL)inside:(Rect2d*)rect {
    return [rect contains:self];
}

#define DOUBLE_TO_BITS(x)  ((Cv64suf){ .f = x }).i

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    int64_t temp = DOUBLE_TO_BITS(self.x);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    temp = DOUBLE_TO_BITS(self.y);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"Point2d {%lf,%lf}", self.x, self.y];
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

