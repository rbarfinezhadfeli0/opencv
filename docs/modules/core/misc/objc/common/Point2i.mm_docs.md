# Documentation for `modules/core/misc/objc/common/Point2i.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Point2i.mm`
- **File Name**: `Point2i.mm`
- **File Size**: 1,911 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Point2i.mm](../../../../../modules/core/misc/objc/common/Point2i.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Point2i.m
//
//  Created by Giles Payne on 2019/10/09.
//

#import "Point2i.h"
#import "Rect2i.h"
#import "CVObjcUtil.h"

@implementation Point2i {
    cv::Point2i native;
}

- (int)x {
    return native.x;
}

- (void)setX:(int)val {
    native.x = val;
}

- (int)y {
    return native.y;
}

- (void)setY:(int)val {
    native.y = val;
}

- (cv::Point2i&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithX:0 y:0];
}

- (instancetype)initWithX:(int)x y:(int)y {
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

+ (instancetype)fromNative:(cv::Point2i&)point {
    return [[Point2i alloc] initWithX:point.x y:point.y];
}

- (void)update:(cv::Point2i&)point {
    self.x = point.x;
    self.y = point.y;
}

- (Point2i*) clone {
    return [[Point2i alloc] initWithX:self.x y:self.y];
}

- (double)dot:(Point2i*)point {
    return self.x * point.x + self.y * point.y;
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.x = (vals != nil && vals.count > 0) ? vals[0].doubleValue : 0;
    self.y = (vals != nil && vals.count > 1) ? vals[1].doubleValue : 0;
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Point2i class]]) {
        return NO;
    } else {
        Point2i* point = (Point2i*)other;
        return self.x == point.x && self.y == point.y;
    }
}

- (BOOL)inside:(Rect2i*)rect {
    return [rect contains:self];
}

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + self.x;
    result = prime * result + self.y;
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"Point2i {%d,%d}", self.x, self.y];
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

