# Documentation for `modules/core/misc/objc/common/TermCriteria.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/TermCriteria.mm`
- **File Name**: `TermCriteria.mm`
- **File Size**: 2,608 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/TermCriteria.mm](../../../../../modules/core/misc/objc/common/TermCriteria.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  TermCriteria.m
//
//  Created by Giles Payne on 2019/12/25.
//

#import "TermCriteria.h"

@implementation TermCriteria {
    cv::TermCriteria native;
}

+ (int)COUNT {
    return 1;
}

+ (int)EPS {
    return 2;
}

+ (int)MAX_ITER {
    return 1;
}

- (int)type {
    return native.type;
}

- (void)setType:(int)val {
    native.type = val;
}

- (int)maxCount {
    return native.maxCount;
}

- (void)setMaxCount:(int)val {
    native.maxCount = val;
}

- (double)epsilon {
    return native.epsilon;
}

- (void)setEpsilon:(double)val {
    native.epsilon = val;
}

#ifdef __cplusplus
- (cv::TermCriteria&)nativeRef {
    return native;
}
#endif

- (instancetype)init {
    return [self initWithType:0 maxCount:0 epsilon:0.0];
}

- (instancetype)initWithType:(int)type maxCount:(int)maxCount epsilon:(double)epsilon {
    self = [super init];
    if (self) {
        self.type = type;
        self.maxCount = maxCount;
        self.epsilon = epsilon;
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

#ifdef __cplusplus
+ (instancetype)fromNative:(cv::TermCriteria&)nativeTermCriteria {
    return [[TermCriteria alloc] initWithType:nativeTermCriteria.type maxCount:nativeTermCriteria.maxCount epsilon:nativeTermCriteria.epsilon];
}
#endif

- (void)set:(NSArray<NSNumber*>*)vals {
    self.type = (vals != nil && vals.count > 0) ? vals[0].intValue : 0;
    self.maxCount = (vals != nil && vals.count > 1) ? vals[1].intValue : 0;
    self.epsilon = (vals != nil && vals.count > 2) ? vals[2].doubleValue : 0.0;
}

- (TermCriteria*)clone {
    return [[TermCriteria alloc] initWithType:self.type maxCount:self.maxCount epsilon:self.epsilon];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[TermCriteria class]]) {
        return NO;
    } else {
        TermCriteria* it = (TermCriteria*)other;
        return self.type == it.type && self.maxCount == it.maxCount && self.epsilon == it.epsilon;
    }
}

#define DOUBLE_TO_BITS(x)  ((Cv64suf){ .f = x }).i

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + self.type;
    result = prime * result + self.maxCount;
    int64_t temp = DOUBLE_TO_BITS(self.epsilon);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"TermCriteria { type: %d, maxCount: %d, epsilon: %lf}", self.type, self.maxCount, self.epsilon];
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

