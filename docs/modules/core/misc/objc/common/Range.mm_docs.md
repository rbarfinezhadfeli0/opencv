# Documentation for `modules/core/misc/objc/common/Range.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Range.mm`
- **File Name**: `Range.mm`
- **File Size**: 2,231 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Range.mm](../../../../../modules/core/misc/objc/common/Range.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Range.mm
//
//  Created by Giles Payne on 2019/10/08.
//

#import "Range.h"

@implementation Range {
    cv::Range native;
}

- (int)start {
    return native.start;
}

- (void)setStart:(int)val {
    native.start = val;
}

- (int)end {
    return native.end;
}

- (void)setEnd:(int)val {
    native.end = val;
}

- (cv::Range&)nativeRef {
    return native;
}

- (instancetype)init {
    return [self initWithStart:0 end: 0];
}

- (instancetype)initWithStart:(int)start end:(int)end {
    self = [super init];
    if (self != nil) {
        self.start = start;
        self.end = end;
    }
    return self;
}

- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [self init];
    if (self != nil) {
        [self set:vals];
    }
    return self;
}

+ (instancetype)fromNative:(cv::Range&)range {
    return [[Range alloc] initWithStart:range.start end:range.end];
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.start = (vals != nil && vals.count > 0) ? vals[0].intValue : 0;
    self.end = (vals != nil && vals.count > 1 ) ? vals[1].intValue : 0;
}

- (int)size {
    return [self empty] ? 0 : self.end - self.start;
}

- (BOOL)empty {
    return self.end <= self.start;
}

+ (Range*)all {
    return [[Range alloc] initWithStart:INT_MIN end:INT_MAX];
}

- (Range*)intersection:(Range*)r1 {
    Range* out = [[Range alloc] initWithStart:MAX(r1.start, self.start) end:MIN(r1.end, self.end)];
    out.end = MAX(out.end, out.start);
    return out;
}

- (Range*)shift:(int)delta {
    return [[Range alloc] initWithStart:self.start + delta end:self.end + delta];
}

- (Range*)clone {
    return [[Range alloc] initWithStart:self.start end:self.end];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[Range class]]) {
        return NO;
    } else {
        Range* it = (Range*)other;
        return self.start == it.start && self.end == it.end;
    }
}

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + self.start;
    result = prime * result + self.end;
    return result;
}

- (NSString *)description {
    return [NSString stringWithFormat:@"Range {%d, %d}", self.start, self.end];
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

