# Documentation for `modules/core/misc/objc/common/Size2i.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Size2i.mm`
- **File Name**: `Size2i.mm`
- **File Size**: 2,160 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Size2i.mm](../../../../../modules/core/misc/objc/common/Size2i.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Size2i.mm
//
//  Created by Giles Payne on 2019/10/06.
//

#import "Size2i.h"
#import "Point2i.h"
#import "CVObjcUtil.h"

@implementation Size2i {
    cv::Size2i native;
}

- (int)width {
    return native.width;
}

- (void)setWidth:(int)val {
    native.width = val;
}

- (int)height {
    return native.height;
}

- (void)setHeight:(int)val {
    native.height = val;
}

- (cv::Size2i&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithWidth:0 height:0];
}

- (instancetype)initWithWidth:(int)width height:(int)height {
    self = [super init];
    if (self) {
        self.width = width;
        self.height = height;
    }
    return self;
}

- (instancetype)initWithPoint:(Point2i*)point {
    return [self initWithWidth:point.x height:point.y];
}

- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [super init];
    if (self) {
        [self set:vals];
    }
    return self;
}

+ (instancetype)fromNative:(cv::Size2i&)size {
    return [[Size2i alloc] initWithWidth:size.width height:size.height];
}

+ (instancetype)width:(int)width height:(int)height {
    return [[Size2i alloc] initWithWidth:width height:height];
}

- (double)area {
    return self.width * self.height;
}

- (BOOL)empty {
    return self.width <= 0 || self.height <= 0;
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.width = (vals != nil && vals.count > 0) ? vals[0].intValue : 0;
    self.height = (vals != nil && vals.count > 1) ? vals[1].intValue : 0;
}

- (Size2i*)clone {
    return [[Size2i alloc] initWithWidth:self.width height:self.height];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Size2i class]]) {
        return NO;
    } else {
        Size2i* it = (Size2i*)other;
        return self.width == it.width && self.height == it.height;
    }
}

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + self.height;
    result = prime * result + self.width;
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"Size2i {%d,%d}", self.width, self.height];
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

