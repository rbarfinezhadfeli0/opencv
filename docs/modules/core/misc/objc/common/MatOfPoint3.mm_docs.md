# Documentation for `modules/core/misc/objc/common/MatOfPoint3.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/MatOfPoint3.mm`
- **File Name**: `MatOfPoint3.mm`
- **File Size**: 2,591 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/MatOfPoint3.mm](../../../../../modules/core/misc/objc/common/MatOfPoint3.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  MatOfPoint3.mm
//
//  Created by Giles Payne on 2019/12/27.
//

#import "MatOfPoint3.h"
#import "Range.h"
#import "Point3i.h"
#import "CvType.h"
#import "ArrayUtil.h"

@implementation MatOfPoint3

static const int _depth = CV_32S;
static const int _channels = 3;

#ifdef __cplusplus
- (instancetype)initWithNativeMat:(cv::Mat*)nativeMat {
    self = [super initWithNativeMat:nativeMat];
    if (self && ![self empty] && [self checkVector:_channels depth:_depth] < 0) {
        @throw [NSException exceptionWithName:NSInvalidArgumentException reason:@"Incompatible Mat" userInfo:nil];
    }
    return self;
}
#endif

- (instancetype)initWithMat:(Mat*)mat {
    self = [super initWithMat:mat rowRange:[Range all]];
    if (self && ![self empty] && [self checkVector:_channels depth:_depth] < 0) {
        @throw [NSException exceptionWithName:NSInvalidArgumentException reason:@"Incompatible Mat" userInfo:nil];
    }
    return self;
}

- (instancetype)initWithArray:(NSArray<Point3i*>*)array {
    self = [super init];
    if (self) {
        [self fromArray:array];
    }
    return self;
}

- (void)alloc:(int)elemNumber {
    if (elemNumber>0) {
        [super create:elemNumber cols:1 type:[CvType makeType:_depth channels:_channels]];
    }
}

- (void)fromArray:(NSArray<Point3i*>*)array {
    NSMutableArray<NSNumber*>* data = [[NSMutableArray alloc] initWithCapacity:array.count * _channels];
    for (int index = 0; index < (int)array.count; index++) {
        data[_channels * index] = [NSNumber numberWithInt:array[index].x];
        data[_channels * index + 1] = [NSNumber numberWithInt:array[index].y];
        data[_channels * index + 2] = [NSNumber numberWithInt:array[index].z];
    }
    [self alloc:(int)array.count];
    [self put:0 col:0 data:data];
}

- (NSArray<Point3i*>*)toArray {
    int length = [self length] / _channels;
    NSMutableArray<Point3i*>* ret = createArrayWithSize(length, [Point3i new]);
    if (length > 0) {
        NSMutableArray<NSNumber*>* data = createArrayWithSize([self length], @0.0);
        [self get:0 col:0 data:data];
        for (int index = 0; index < length; index++) {
            ret[index] = [[Point3i alloc] initWithX:data[index * _channels].intValue y:data[index * _channels + 1].intValue z:data[index * _channels + 2].intValue];
        }
    }
    return ret;
}

- (int)length {
    int num = [self checkVector:_channels depth:_depth];
    if (num < 0) {
        @throw  [NSException exceptionWithName:NSInternalInconsistencyException reason:@"Incompatible Mat" userInfo:nil];
    }
    return num * _channels;
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

