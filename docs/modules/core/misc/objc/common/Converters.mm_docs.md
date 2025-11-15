# Documentation for `modules/core/misc/objc/common/Converters.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/Converters.mm`
- **File Name**: `Converters.mm`
- **File Size**: 6,654 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/Converters.mm](../../../../../modules/core/misc/objc/common/Converters.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  Converters.mm
//
//  Created by  Giles Payne on 31/05/2020.
//

#import "Converters.h"
#import "ArrayUtil.h"
#import "MatOfPoint2i.h"
#import "MatOfPoint2f.h"
#import "MatOfPoint3.h"
#import "MatOfPoint3f.h"
#import "MatOfFloat.h"
#import "MatOfByte.h"
#import "MatOfInt.h"
#import "MatOfDouble.h"
#import "MatOfRect2i.h"
#import "MatOfRect2d.h"
#import "MatOfKeyPoint.h"
#import "MatOfDMatch.h"
#import "MatOfRotatedRect.h"

@implementation Converters

+ (Mat*)vector_Point_to_Mat:(NSArray<Point2i*>*)pts {
    return [[MatOfPoint2i alloc] initWithArray:pts];
}

+ (NSArray<Point2i*>*)Mat_to_vector_Point:(Mat*)mat {
    return [[[MatOfPoint2i alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_Point2f_to_Mat:(NSArray<Point2f*>*)pts {
    return [[MatOfPoint2f alloc] initWithArray:pts];
}

+ (NSArray<Point2f*>*)Mat_to_vector_Point2f:(Mat*)mat {
    return [[[MatOfPoint2f alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_Point2d_to_Mat:(NSArray<Point2d*>*)pts {
    Mat* res = [[Mat alloc] initWithRows:(int)pts.count cols:1 type:CV_64FC2];
    NSMutableArray<NSNumber*>* buff = [NSMutableArray arrayWithCapacity:pts.count*2];
    for (Point2d* pt in pts) {
        [buff addObject:[NSNumber numberWithDouble:pt.x]];
        [buff addObject:[NSNumber numberWithDouble:pt.y]];
    }
    [res put:0 col:0 data:buff];
    return res;
}

+ (NSArray<Point2d*>*)Mat_to_vector_Point2d:(Mat*)mat {
    if (mat.cols != 1 || mat.type != CV_64FC2) {
        NSException* exception = [NSException
                exceptionWithName:@"UnsupportedOperationException"
                                  reason:[NSString stringWithFormat:@"Invalid Mat. Mat must be of type CV_64FC2 and have 1 column."]
                userInfo:nil];
        @throw exception;
    }
    NSMutableArray<Point2d*>* ret = [NSMutableArray new];
    NSMutableArray<NSNumber*>* buff = createArrayWithSize(mat.rows*2, [NSNumber numberWithInt:0]);
    [mat get:0 col:0 data:buff];
    for (int i = 0; i < mat.rows; i++) {
        [ret addObject:[[Point2d alloc] initWithX:buff[i * 2].doubleValue y:buff[i * 2 + 1].doubleValue]];
    }
    return ret;
}

+ (Mat*)vector_Point3i_to_Mat:(NSArray<Point3i*>*)pts {
    return [[MatOfPoint3 alloc] initWithArray:pts];
}

+ (NSArray<Point3i*>*)Mat_to_vector_Point3i:(Mat*)mat {
    return [[[MatOfPoint3 alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_Point3f_to_Mat:(NSArray<Point3f*>*)pts {
    return [[MatOfPoint3f alloc] initWithArray:pts];
}

+ (NSArray<Point3f*>*)Mat_to_vector_Point3f:(Mat*)mat {
    return [[[MatOfPoint3f alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_Point3d_to_Mat:(NSArray<Point3d*>*)pts {
    Mat* res = [[Mat alloc] initWithRows:(int)pts.count cols:1 type:CV_64FC3];
    NSMutableArray<NSNumber*>* buff = [NSMutableArray arrayWithCapacity:pts.count*3];
    for (Point3d* pt in pts) {
        [buff addObject:[NSNumber numberWithDouble:pt.x]];
        [buff addObject:[NSNumber numberWithDouble:pt.y]];
        [buff addObject:[NSNumber numberWithDouble:pt.z]];
    }
    [res put:0 col:0 data:buff];
    return res;
}

+ (NSArray<Point3d*>*)Mat_to_vector_Point3d:(Mat*)mat {
    if (mat.cols != 1 || mat.type != CV_64FC3) {
        NSException* exception = [NSException
                exceptionWithName:@"UnsupportedOperationException"
                                  reason:[NSString stringWithFormat:@"Invalid Mat. Mat must be of type CV_64FC3 and have 1 column."]
                userInfo:nil];
        @throw exception;
    }
    NSMutableArray<Point3d*>* ret = [NSMutableArray new];
    NSMutableArray<NSNumber*>* buff = createArrayWithSize(mat.rows*3, [NSNumber numberWithInt:0]);
    [mat get:0 col:0 data:buff];
    for (int i = 0; i < mat.rows; i++) {
        [ret addObject:[[Point3d alloc] initWithX:buff[i * 3].doubleValue y:buff[i * 3 + 1].doubleValue z:buff[i * 3 + 2].doubleValue]];
    }
    return ret;
}

+ (Mat*)vector_float_to_Mat:(NSArray<NSNumber*>*)fs {
    return [[MatOfFloat alloc] initWithArray:fs];
}

+ (NSArray<NSNumber*>*)Mat_to_vector_float:(Mat*)mat {
    return [[[MatOfFloat alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_uchar_to_Mat:(NSArray<NSNumber*>*)us {
    return [[MatOfByte alloc] initWithArray:us];
}

+ (NSArray<NSNumber*>*)Mat_to_vector_uchar:(Mat*)mat {
    return [[[MatOfByte alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_char_to_Mat:(NSArray<NSNumber*>*)cs {
    Mat* res = [[Mat alloc] initWithRows:(int)cs.count cols:1 type:CV_8S];
    [res put:0 col:0 data:cs];
    return res;
}

+ (NSArray<NSNumber*>*)Mat_to_vector_char:(Mat*)mat {
    if (mat.cols != 1 || mat.type != CV_8S) {
        NSException* exception = [NSException
                exceptionWithName:@"UnsupportedOperationException"
                                  reason:[NSString stringWithFormat:@"Invalid Mat. Mat must be of type CV_8S and have 1 column."]
                userInfo:nil];
        @throw exception;
    }
    NSMutableArray<NSNumber*>* ret = createArrayWithSize(mat.rows, @0);
    [mat get:0 col:0 data:ret];
    return ret;
}

+ (Mat*)vector_int_to_Mat:(NSArray<NSNumber*>*)is {
    return [[MatOfInt alloc] initWithArray:is];
}

+ (NSArray<NSNumber*>*)Mat_to_vector_int:(Mat*)mat {
    return [[[MatOfInt alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_Rect_to_Mat:(NSArray<Rect2i*>*)rs {
    return [[MatOfRect2i alloc] initWithArray:rs];
}

+ (NSArray<Rect2i*>*)Mat_to_vector_Rect:(Mat*)mat {
    return [[[MatOfRect2i alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_Rect2d_to_Mat:(NSArray<Rect2d*>*)rs {
    return [[MatOfRect2d alloc] initWithArray:rs];
}

+ (NSArray<Rect2d*>*)Mat_to_vector_Rect2d:(Mat*)mat {
    return [[[MatOfRect2d alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_KeyPoint_to_Mat:(NSArray<KeyPoint*>*)kps {
    return [[MatOfKeyPoint alloc] initWithArray:kps];
}

+ (NSArray<KeyPoint*>*)Mat_to_vector_KeyPoint:(Mat*)mat {
    return [[[MatOfKeyPoint alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_double_to_Mat:(NSArray<NSNumber*>*)ds {
    return [[MatOfDouble alloc] initWithArray:ds];
}

+ (NSArray<NSNumber*>*)Mat_to_vector_double:(Mat*)mat {
    return [[[MatOfDouble alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_DMatch_to_Mat:(NSArray<DMatch*>*)matches {
    return [[MatOfDMatch alloc] initWithArray:matches];
}

+ (NSArray<DMatch*>*)Mat_to_vector_DMatch:(Mat*)mat {
    return [[[MatOfDMatch alloc] initWithMat:mat] toArray];
}

+ (Mat*)vector_RotatedRect_to_Mat:(NSArray<RotatedRect*>*)rs {
    return [[MatOfRotatedRect alloc] initWithArray:rs];
}

+ (NSArray<RotatedRect*>*)Mat_to_vector_RotatedRect:(Mat*)mat {
    return [[[MatOfRotatedRect alloc] initWithMat:mat] toArray];
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

