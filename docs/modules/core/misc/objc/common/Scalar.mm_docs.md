# Documentation for `modules/core/misc/objc/common/Scalar.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Scalar.mm`
- **File Name**: `Scalar.mm`
- **File Size**: 3,509 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Scalar.mm](../../../../../modules/core/misc/objc/common/Scalar.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Scalar.mm
//
//  Created by Giles Payne on 2019/10/06.
//

#import "Scalar.h"

double getVal(NSArray<NSNumber*>* vals, int index) {
    return [vals count] > index ? vals[index].doubleValue : 0;
}

@implementation Scalar {
    cv::Scalar native;
}

- (NSArray<NSNumber*>*)val {
    return @[[NSNumber numberWithDouble:native.val[0]], [NSNumber numberWithDouble:native.val[1]], [NSNumber numberWithDouble:native.val[2]], [NSNumber numberWithDouble:native.val[3]]];
}

#ifdef __cplusplus
- (cv::Scalar&)nativeRef {
    return native;
}
#endif

- (instancetype)initWithVals:(NSArray<NSNumber*> *)vals {
    return [self initWithV0:getVal(vals, 0) v1:getVal(vals, 1) v2:getVal(vals, 2) v3:getVal(vals, 3)];
}

- (instancetype)initWithV0:(double)v0 v1:(double)v1 v2:(double)v2 v3:(double)v3 {
    self = [super init];
    if (self != nil) {
        native.val[0] = v0;
        native.val[1] = v1;
        native.val[2] = v2;
        native.val[3] = v3;
    }
    return self;
}

- (instancetype)initWithV0:(double)v0 v1:(double)v1 v2:(double)v2 {
    return [self initWithV0:v0 v1:v1 v2:v2 v3:0];
}

- (instancetype)initWithV0:(double)v0 v1:(double)v1 {
    return [self initWithV0:v0 v1:v1 v2:0 v3:0];
}

- (instancetype)initWithV0:(double)v0 {
    return [self initWithV0:v0 v1:0 v2:0 v3:0];
}

#ifdef __cplusplus
+ (instancetype)fromNative:(cv::Scalar&)nativeScalar {
    return [[Scalar alloc] initWithV0:nativeScalar.val[0] v1:nativeScalar.val[1] v2:nativeScalar.val[2] v3:nativeScalar.val[3]];
}
#endif

+ (Scalar*)all:(double)v {
    return [[Scalar alloc] initWithV0:v v1:v v2:v v3:v];
}

- (Scalar*)clone {
    return [Scalar fromNative:self.nativeRef];
}

- (Scalar*)mul:(Scalar*)it scale:(double)scale {
    return [[Scalar alloc] initWithV0:self.nativeRef.val[0]*it.nativeRef.val[0]*scale v1:self.nativeRef.val[1]*it.nativeRef.val[1]*scale v2:self.nativeRef.val[2]*it.nativeRef.val[2]*scale v3:self.nativeRef.val[3]*it.nativeRef.val[3]*scale];
}

- (Scalar*)mul:(Scalar*)it {
    return [self mul:it scale:1];
}

- (Scalar*)conj {
    return [[Scalar alloc] initWithV0:self.nativeRef.val[0] v1:-self.nativeRef.val[1] v2:-self.nativeRef.val[2] v3:-self.nativeRef.val[3]];
}

- (BOOL)isReal {
    return self.nativeRef.val[1] == self.nativeRef.val[2] == self.nativeRef.val[3] == 0;
}

- (BOOL)isEqual:(id)other
{
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Scalar class]]) {
        return NO;
    } else {
        Scalar* it = (Scalar*) other;
        return it.nativeRef.val[0] == self.nativeRef.val[0] && it.nativeRef.val[1] == self.nativeRef.val[1] && it.nativeRef.val[2] == self.nativeRef.val[2] && it.nativeRef.val[3] == self.nativeRef.val[3];
    }
}

#define DOUBLE_TO_BITS(x)  ((Cv64suf){ .f = x }).i

- (NSUInteger)hash
{
    int prime = 31;
    uint32_t result = 1;
    int64_t temp = DOUBLE_TO_BITS(self.nativeRef.val[0]);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    temp = DOUBLE_TO_BITS(self.nativeRef.val[1]);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    temp = DOUBLE_TO_BITS(self.nativeRef.val[2]);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    temp = DOUBLE_TO_BITS(self.nativeRef.val[3]);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    return result;
}

- (NSString *)description
{
    return [NSString stringWithFormat:@"Scalar [%lf, %lf, %lf, %lf]", self.nativeRef.val[0], self.nativeRef.val[1], self.nativeRef.val[2], self.nativeRef.val[3]];
}

@end
```

## High-Level Overview

This is a .mm source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

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

