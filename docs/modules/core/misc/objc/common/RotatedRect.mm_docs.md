# Documentation for `modules/core/misc/objc/common/RotatedRect.mm`

## File Metadata

- **Full Path**: `modules/core/misc/objc/common/RotatedRect.mm`
- **File Name**: `RotatedRect.mm`
- **File Size**: 4,042 bytes
- **File Type**: .mm
- **Link to Source**: [modules/core/misc/objc/common/RotatedRect.mm](../../../../../modules/core/misc/objc/common/RotatedRect.mm)

## Purpose and Role

This file is located in the `modules/core/misc/objc/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  RotatedRect.m
//
//  Created by Giles Payne on 2019/12/26.
//

#import "RotatedRect.h"
#import "Point2f.h"
#import "Size2f.h"
#import "Rect2f.h"

#include <math.h>

@implementation RotatedRect {
    cv::RotatedRect native;
}

- (cv::RotatedRect&)nativeRef {
    native.center.x = self.center.x;
    native.center.y = self.center.y;
    native.size.width = self.size.width;
    native.size.height = self.size.height;
    native.angle = self.angle;
    return native;
}

- (instancetype)init {
    return [self initWithCenter:[Point2f new] size:[Size2f new] angle:0.0];
}

- (instancetype)initWithCenter:(Point2f*)center size:(Size2f*)size angle:(double)angle {
    self = [super init];
    if (self) {
        self.center = center;
        self.size = size;
        self.angle = angle;
    }
    return self;
}

- (instancetype)initWithVals:(NSArray<NSNumber*>*)vals {
    self = [self init];
    if (self) {
        [self set:vals];
    }
    return self;
}

+ (instancetype)fromNative:(cv::RotatedRect&)rotatedRect {
    return [[RotatedRect alloc] initWithCenter:[Point2f fromNative:rotatedRect.center] size:[Size2f fromNative:rotatedRect.size] angle:rotatedRect.angle];
}

- (void)set:(NSArray<NSNumber*>*)vals {
    self.center.x = (vals != nil && vals.count > 0) ? vals[0].floatValue : 0.0;
    self.center.y = (vals != nil && vals.count > 1) ? vals[1].floatValue : 0.0;
    self.size.width = (vals != nil && vals.count > 2) ? vals[2].floatValue : 0.0;
    self.size.height = (vals != nil && vals.count > 3) ? vals[3].floatValue : 0.0;
    self.angle = (vals != nil && vals.count > 4) ? vals[4].doubleValue : 0.0;
}

- (NSArray<Point2f*>*)points {
    double angleRadians = self.angle * M_PI / 180.0;
    double b = cos(angleRadians) * 0.5;
    double a = sin(angleRadians) * 0.5f;

    Point2f* p0 = [[Point2f alloc] initWithX:self.center.x - a * self.size.height - b * self.size.width y:self.center.y + b * self.size.height - a * self.size.width];
    Point2f* p1 = [[Point2f alloc] initWithX:self.center.x + a * self.size.height - b * self.size.width y:self.center.y - b * self.size.height - a * self.size.width];
    Point2f* p2 = [[Point2f alloc] initWithX:2 * self.center.x - p0.x y:2 * self.center.y - p0.y];
    Point2f* p3 = [[Point2f alloc] initWithX:2 * self.center.x - p1.x y:2 * self.center.y - p1.y];
    return [NSArray arrayWithObjects:p0, p1, p2, p3, nil];
}

- (Rect2f*)boundingRect {
    NSArray<Point2f*>* pts = [self points];
    Rect2f* rect = [[Rect2f alloc] initWithX:(int)floor(MIN(MIN(MIN(pts[0].x, pts[1].x), pts[2].x), pts[3].x)) y:(int)floor(MIN(MIN(MIN(pts[0].y, pts[1].y), pts[2].y), pts[3].y)) width:(int)ceil(MAX(MAX(MAX(pts[0].x, pts[1].x), pts[2].x), pts[3].x)) height:(int)ceil(MAX(MAX(MAX(pts[0].y, pts[1].y), pts[2].y), pts[3].y))];
    rect.width -= rect.x - 1;
    rect.height -= rect.y - 1;
    return rect;
}

- (RotatedRect*)clone {
    return [[RotatedRect alloc] initWithCenter:[self.center clone] size:[self.size clone] angle:self.angle];
}

- (BOOL)isEqual:(id)other {
    if (other == self) {
        return YES;
    } else if (![other isKindOfClass:[RotatedRect class]]) {
        return NO;
    } else {
        RotatedRect* rect = (RotatedRect*)other;
        return [self.center isEqual:rect.center] && [self.size isEqual:rect.size] && self.angle == rect.angle;
    }
}

#define FLOAT_TO_BITS(x)  ((Cv32suf){ .f = x }).i
#define DOUBLE_TO_BITS(x)  ((Cv64suf){ .f = x }).i

- (NSUInteger)hash {
    int prime = 31;
    uint32_t result = 1;
    result = prime * result + FLOAT_TO_BITS(self.center.x);
    result = prime * result + FLOAT_TO_BITS(self.center.y);
    result = prime * result + FLOAT_TO_BITS(self.size.width);
    result = prime * result + FLOAT_TO_BITS(self.size.height);
    int64_t temp = DOUBLE_TO_BITS(self.angle);
    result = prime * result + (int32_t) (temp ^ (temp >> 32));
    return result;
}

- (NSString*)description {
    return [NSString stringWithFormat:@"RotatedRect {%@,%@,%lf}", self.center.description, self.size.description, self.angle];
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

**C++ Includes:**
- `math.h`


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

