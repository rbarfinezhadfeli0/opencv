# Documentation for `modules/core/misc/java/src/java/core+TermCriteria.java`

## File Metadata

- **Full Path**: `modules/core/misc/java/src/java/core+TermCriteria.java`
- **File Name**: `core+TermCriteria.java`
- **File Size**: 2,607 bytes
- **File Type**: .java
- **Link to Source**: [modules/core/misc/java/src/java/core+TermCriteria.java](../../../../../../modules/core/misc/java/src/java/core+TermCriteria.java)

## Purpose and Role

This file is located in the `modules/core/misc/java/src/java` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.core;

//javadoc:TermCriteria
public class TermCriteria {

    /**
     * The maximum number of iterations or elements to compute
     */
    public static final int COUNT = 1;
    /**
     * The maximum number of iterations or elements to compute
     */
    public static final int MAX_ITER = COUNT;
    /**
     * The desired accuracy threshold or change in parameters at which the iterative algorithm is terminated.
     */
    public static final int EPS = 2;

    public int type;
    public int maxCount;
    public double epsilon;

    /**
     * Termination criteria for iterative algorithms.
     *
     * @param type
     *            the type of termination criteria: COUNT, EPS or COUNT + EPS.
     * @param maxCount
     *            the maximum number of iterations/elements.
     * @param epsilon
     *            the desired accuracy.
     */
    public TermCriteria(int type, int maxCount, double epsilon) {
        this.type = type;
        this.maxCount = maxCount;
        this.epsilon = epsilon;
    }

    /**
     * Termination criteria for iterative algorithms.
     */
    public TermCriteria() {
        this(0, 0, 0.0);
    }

    public TermCriteria(double[] vals) {
        set(vals);
    }

    public void set(double[] vals) {
        if (vals != null) {
            type = vals.length > 0 ? (int) vals[0] : 0;
            maxCount = vals.length > 1 ? (int) vals[1] : 0;
            epsilon = vals.length > 2 ? (double) vals[2] : 0;
        } else {
            type = 0;
            maxCount = 0;
            epsilon = 0;
        }
    }

    public TermCriteria clone() {
        return new TermCriteria(type, maxCount, epsilon);
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        long temp;
        temp = Double.doubleToLongBits(type);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        temp = Double.doubleToLongBits(maxCount);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        temp = Double.doubleToLongBits(epsilon);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof TermCriteria)) return false;
        TermCriteria it = (TermCriteria) obj;
        return type == it.type && maxCount == it.maxCount && epsilon == it.epsilon;
    }

    @Override
    public String toString() {
        return "{ type: " + type + ", maxCount: " + maxCount + ", epsilon: " + epsilon + "}";
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

- **TermCriteria**: A class/struct defined in this file


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

