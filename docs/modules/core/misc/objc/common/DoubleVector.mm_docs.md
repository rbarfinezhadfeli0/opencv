# Documentation for `modules/core/misc/objc/common/DoubleVector.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/DoubleVector.mm`
- **File Name**: `DoubleVector.mm`
- **File Size**: 1,745 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/DoubleVector.mm](../../../../../modules/core/misc/objc/common/DoubleVector.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  DoubleVector.m
//
//  Created by Giles Payne on 2020/01/04.
//

#import "DoubleVector.h"
#import <vector>

@implementation DoubleVector {
    std::vector<double> v;
}

-(instancetype)initWithData:(NSData*)data {
    self = [super init];
    if (self) {
        if (data.length % sizeof(double) != 0) {
            @throw [NSException exceptionWithName:NSInvalidArgumentException reason:@"Invalid data length" userInfo:nil];
        }
        v.insert(v.begin(), (double*)data.bytes, (double*)data.bytes + data.length/sizeof(double));
    }
    return self;
}

-(instancetype)initWithVector:(DoubleVector*)src {
    self = [super init];
    if (self) {
        v.insert(v.begin(), src.nativeRef.begin(), src.nativeRef.end());
    }
    return self;
}

-(size_t)length {
    return v.size();
}

-(double*)nativeArray {
    return (double*)v.data();
}

-(instancetype)initWithNativeArray:(double*)array elements:(int)elements {
    self = [super init];
    if (self) {
        v.insert(v.begin(), array, array + elements);
    }
    return self;
}

- (std::vector<double>&)nativeRef {
    return v;
}

-(instancetype)initWithStdVector:(std::vector<double>&)src {
    self = [super init];
    if (self) {
        v.insert(v.begin(), src.begin(), src.end());
    }
    return self;
}

+(instancetype)fromNative:(std::vector<double>&)src {
    return [[DoubleVector alloc] initWithStdVector:src];
}

-(double)get:(NSInteger)index {
    if (index < 0 || index >= (long)v.size()) {
        @throw [NSException exceptionWithName:NSRangeException reason:@"Invalid data length" userInfo:nil];
    }
    return v[index];
}

-(NSData*)data {
    return [NSData dataWithBytesNoCopy:v.data() length:(v.size() * sizeof(double)) freeWhenDone:NO];
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

