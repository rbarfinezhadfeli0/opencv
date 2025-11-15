# Documentation for `modules/core/misc/objc/common/Double2.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Double2.mm`
- **File Name**: `Double2.mm`
- **File Size**: 1,389 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Double2.mm](../../../../../modules/core/misc/objc/common/Double2.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Double2.mm
//
//  Created by Giles Payne on 2020/05/22.
//

#import "Double2.h"
#import "Mat.h"

@implementation Double2 {
    cv::Vec2d native;
}

-(double)v0 {
    return native[0];
}

-(void)setV0:(double)v {
    native[0] = v;
}

-(double)v1 {
    return native[1];
}

-(void)setV1:(double)v {
    native[1] = v;
}

-(instancetype)init {
    return [self initWithV0:0 v1:0];
}

-(instancetype)initWithV0:(double)v0 v1:(double)v1 {
    self = [super init];
    if (self) {
        self.v0 = v0;
        self.v1 = v1;
    }
    return self;
}

-(instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [super init];
    if (self) {
        [self set:vals];
    }
    return self;
}

+(instancetype)fromNative:(cv::Vec2d&)vec2d {
    return [[Double2 alloc] initWithV0:vec2d[0] v1:vec2d[1]];
}

-(void)set:(NSArray<NSNumber*>*)vals {
    self.v0 = (vals != nil && vals.count > 0) ? vals[0].doubleValue : 0;
    self.v1 = (vals != nil && vals.count > 1) ? vals[1].doubleValue : 0;
}

-(NSArray<NSNumber*>*)get {
    return @[[NSNumber numberWithFloat:native[0]], [NSNumber numberWithFloat:native[1]]];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Double2 class]]) {
        return NO;
    } else {
        Double2* d2 = (Double2*)other;
        return self.v0 == d2.v0 && self.v1 == d2.v1;
    }
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

