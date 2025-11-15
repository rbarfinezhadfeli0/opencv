# Documentation for `modules/core/misc/objc/common/Point3d.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Point3d.mm`
- **File Name**: `Point3d.mm`
- **File Size**: 2,714 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Point3d.mm](../../../../../modules/core/misc/objc/common/Point3d.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Point3d.mm
//
//  Created by Giles Payne on 2019/10/09.
//

#import "Point3d.h"
#import "Point2d.h"

@implementation Point3d {
    cv::Point3d native;
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

- (double)z {
    return native.z;
}

- (void)setZ:(double)val {
    native.z = val;
}

- (cv::Point3d&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithX:0 y:0 z:0];
}

- (instancetype)initWithX:(double)x y:(double)y z:(double)z {
    self = [super init];
    if (self) {
        self.x = x;
        self.y = y;
        self.z = z;
    }
    return self;
}

- (instancetype)initWithPoint:(Point2d*)point {
    return [self initWithX:point.x y:point.y z:0];
}

- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [super init];
    if (self) {
        [self set:vals];
    }
    return self;
}

+ (instancetype)fromNative:(cv::Point3d&)point {
    return [[Point3d alloc] initWithX:point.x y:point.y z:point.z];
}

- (void)update:(cv::Point3d&)point {
    self.x = point.x;
    self.y = point.y;
    self.z = point.z;
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.x = (vals != nil && vals.count > 0) ? vals[0].doubleValue : 0.0;
    self.y = (vals != nil && vals.count > 1) ? vals[1].doubleValue : 0.0;
    self.z = (vals != nil && vals.count > 2) ? vals[2].doubleValue : 0.0;
}

- (Point3d*) clone {
    return [[Point3d alloc] initWithX:self.x y:self.y z:self.z];
}

- (double)dot:(Point3d*)point {
    return self.x * point.x + self.y * point.y + self.z * point.z;
}

- (Point3d*)cross:(Point3d*)point {
    return [[Point3d alloc] initWithX:(self.y * point.z - self.z * point.y) y:(self.z * point.x - self.x * point.z) z:(self.x * point.y - self.y * point.x)];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Point3d class]]) {
        return NO;
    } else {
        Point3d* point = (Point3d*)other;
        return self.x == point.x && self.y == point.y && self.z == point.z;
    }
}

#define DOUBLE_TO_BITS(x)  ((Cv64suf){ .f = x }).i

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    int64_t temp = DOUBLE_TO_BITS(self.x);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    temp = DOUBLE_TO_BITS(self.y);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    temp = DOUBLE_TO_BITS(self.z);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"Point3 {%lf,%lf,%lf}", self.x, self.y, self.z];
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

