# Documentation for `modules/core/misc/objc/common/Point3f.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Point3f.mm`
- **File Name**: `Point3f.mm`
- **File Size**: 2,556 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Point3f.mm](../../../../../modules/core/misc/objc/common/Point3f.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Point3f.mm
//
//  Created by Giles Payne on 2019/10/09.
//

#import "Point3f.h"
#import "Point2f.h"

@implementation Point3f {
    cv::Point3f native;
}

- (float)x {
    return native.x;
}

- (void)setX:(float)val {
    native.x = val;
}

- (float)y {
    return native.y;
}

- (void)setY:(float)val {
    native.y = val;
}

- (float)z {
    return native.z;
}

- (void)setZ:(float)val {
    native.z = val;
}

- (cv::Point3f&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithX:0 y:0 z:0];
}

- (instancetype)initWithX:(float)x y:(float)y z:(float)z {
    self = [super init];
    if (self) {
        self.x = x;
        self.y = y;
        self.z = z;
    }
    return self;
}

- (instancetype)initWithPoint:(Point2f*)point {
    return [self initWithX:point.x y:point.y z:0];
}

- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [super init];
    if (self) {
        [self set:vals];
    }
    return self;
}

+ (instancetype)fromNative:(cv::Point3f&)point {
    return [[Point3f alloc] initWithX:point.x y:point.y z:point.z];
}

- (void)update:(cv::Point3f&)point {
    self.x = point.x;
    self.y = point.y;
    self.z = point.z;
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.x = (vals != nil && vals.count > 0) ? vals[0].floatValue : 0.0;
    self.y = (vals != nil && vals.count > 1) ? vals[1].floatValue : 0.0;
    self.z = (vals != nil && vals.count > 2) ? vals[2].floatValue : 0.0;
}

- (Point3f*) clone {
    return [[Point3f alloc] initWithX:self.x y:self.y z:self.z];
}

- (double)dot:(Point3f*)point {
    return self.x * point.x + self.y * point.y + self.z * point.z;
}

- (Point3f*)cross:(Point3f*)point {
    return [[Point3f alloc] initWithX:(self.y * point.z - self.z * point.y) y:(self.z * point.x - self.x * point.z) z:(self.x * point.y - self.y * point.x)];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Point3f class]]) {
        return NO;
    } else {
        Point3f* point = (Point3f*)other;
        return self.x == point.x && self.y == point.y && self.z == point.z;
    }
}

#define FLOAT_TO_BITS(x)  ((Cv32suf){ .f = x }).i

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + FLOAT_TO_BITS(self.x);
    result = prime * result + FLOAT_TO_BITS(self.y);
    result = prime * result + FLOAT_TO_BITS(self.z);
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"Point3f {%f,%f,%f}", self.x, self.y, self.z];
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

