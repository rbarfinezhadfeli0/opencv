# Documentation for `modules/core/misc/java/src/java/core+RotatedRect.java`

## File Metadata

- **Full Path**: `modules/core/misc/java/src/java/core+RotatedRect.java`
- **File Name**: `core+RotatedRect.java`
- **File Size**: 3,573 bytes
- **File Type**: .java
- **Link to Source**: [modules/core/misc/java/src/java/core+RotatedRect.java](../../../../../../modules/core/misc/java/src/java/core+RotatedRect.java)

## Purpose and Role

This file is located in the `modules/core/misc/java/src/java` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.core;

//javadoc:RotatedRect_
public class RotatedRect {

    public Point center;
    public Size size;
    public double angle;

    public RotatedRect() {
        this.center = new Point();
        this.size = new Size();
        this.angle = 0;
    }

    public RotatedRect(Point c, Size s, double a) {
        this.center = c.clone();
        this.size = s.clone();
        this.angle = a;
    }

    public RotatedRect(double[] vals) {
        this();
        set(vals);
    }

    public void set(double[] vals) {
        if (vals != null) {
            center.x = vals.length > 0 ? (double) vals[0] : 0;
            center.y = vals.length > 1 ? (double) vals[1] : 0;
            size.width = vals.length > 2 ? (double) vals[2] : 0;
            size.height = vals.length > 3 ? (double) vals[3] : 0;
            angle = vals.length > 4 ? (double) vals[4] : 0;
        } else {
            center.x = 0;
            center.y = 0;
            size.width = 0;
            size.height = 0;
            angle = 0;
        }
    }

    public void points(Point pt[])
    {
        double _angle = angle * Math.PI / 180.0;
        double b = (double) Math.cos(_angle) * 0.5f;
        double a = (double) Math.sin(_angle) * 0.5f;

        pt[0] = new Point(
                center.x - a * size.height - b * size.width,
                center.y + b * size.height - a * size.width);

        pt[1] = new Point(
                center.x + a * size.height - b * size.width,
                center.y - b * size.height - a * size.width);

        pt[2] = new Point(
                2 * center.x - pt[0].x,
                2 * center.y - pt[0].y);

        pt[3] = new Point(
                2 * center.x - pt[1].x,
                2 * center.y - pt[1].y);
    }

    public Rect boundingRect()
    {
        Point pt[] = new Point[4];
        points(pt);
        Rect r = new Rect((int) Math.floor(Math.min(Math.min(Math.min(pt[0].x, pt[1].x), pt[2].x), pt[3].x)),
                (int) Math.floor(Math.min(Math.min(Math.min(pt[0].y, pt[1].y), pt[2].y), pt[3].y)),
                (int) Math.ceil(Math.max(Math.max(Math.max(pt[0].x, pt[1].x), pt[2].x), pt[3].x)),
                (int) Math.ceil(Math.max(Math.max(Math.max(pt[0].y, pt[1].y), pt[2].y), pt[3].y)));
        r.width -= r.x - 1;
        r.height -= r.y - 1;
        return r;
    }

    public RotatedRect clone() {
        return new RotatedRect(center, size, angle);
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        long temp;
        temp = Double.doubleToLongBits(center.x);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        temp = Double.doubleToLongBits(center.y);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        temp = Double.doubleToLongBits(size.width);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        temp = Double.doubleToLongBits(size.height);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        temp = Double.doubleToLongBits(angle);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof RotatedRect)) return false;
        RotatedRect it = (RotatedRect) obj;
        return center.equals(it.center) && size.equals(it.size) && angle == it.angle;
    }

    @Override
    public String toString() {
        return "{ " + center + " " + size + " * " + angle + " }";
    }
}
```

## High-Level Overview

This is a Java file that provides Java bindings or Android support for OpenCV.

**Key Characteristics:**
- Implements Java API for OpenCV functionality
- May use JNI to interface with native code
- Follows Java coding conventions
- Part of the OpenCV Java/Android SDK


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **RotatedRect**: A class/struct defined in this file


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

