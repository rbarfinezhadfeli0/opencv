# Documentation for `modules/gapi/include/opencv2/gapi/gcompiled.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gcompiled.hpp`
- **File Name**: `gcompiled.hpp`
- **File Size**: 8,077 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gcompiled.hpp](../../../../../modules/gapi/include/opencv2/gapi/gcompiled.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#ifndef OPENCV_GAPI_GCOMPILED_HPP
#define OPENCV_GAPI_GCOMPILED_HPP

#include <vector>

#include <opencv2/gapi/opencv_includes.hpp>
#include <opencv2/gapi/own/assert.hpp>
#include <opencv2/gapi/garg.hpp>

namespace cv {

// This class represents a compiled computation.
// In theory (and ideally), it can be used w/o the rest of APIs.
// In theory (and ideally), it can be serialized/deserialized.
// It can enable scenarious like deployment to an autonomous devince, FuSa, etc.
//
// Currently GCompiled assumes all GMats you used to pass data to G-API
// are valid and not destroyed while you use a GCompiled object.
//
// FIXME: In future, there should be a way to name I/O objects and specify it
// to GCompiled externally (for example, when it is loaded on the target system).

/**
 * \addtogroup gapi_main_classes
 * @{
 */
/**
 * @brief Represents a compiled computation (graph). Can only be used
 * with image / data formats & resolutions it was compiled for, with
 * some exceptions.
 *
 * This class represents a product of graph compilation (calling
 * cv::GComputation::compile()). Objects of this class actually do
 * data processing, and graph execution is incapsulated into objects
 * of this class. Execution model itself depends on kernels and
 * backends which were using during the compilation, see @ref
 * gapi_compile_args for details.
 *
 * In a general case, GCompiled objects can be applied to data only in
 * that formats/resolutions they were compiled for (see @ref
 * gapi_meta_args). However, if the underlying backends allow, a
 * compiled object can be _reshaped_ to handle data (images) of
 * different resolution, though formats and types must remain the same.
 *
 * GCompiled is very similar to `std::function<>` in its semantics --
 * running it looks like a function call in the user code.
 *
 * At the moment, GCompiled objects are not reentrant -- generally,
 * the objects are stateful since graph execution itself is a stateful
 * process and this state is now maintained in GCompiled's own memory
 * (not on the process stack).
 *
 * At the same time, two different GCompiled objects produced from the
 * single cv::GComputation are completely independent and can be used
 * concurrently.
 *
 * @sa GStreamingCompiled
 */
class GAPI_EXPORTS GCompiled
{
public:
    /// @private
    class GAPI_EXPORTS Priv;

    /**
     * @brief Constructs an empty object
     */
    GCompiled();

    /**
     * @brief Run the compiled computation, a generic version.
     *
     * @param ins vector of inputs to process.
     * @param outs vector of outputs to produce.
     *
     * Input/output vectors must have the same number of elements as
     * defined in the cv::GComputation protocol (at the moment of its
     * construction). Shapes of elements also must conform to protocol
     * (e.g. cv::Mat needs to be passed where cv::GMat has been
     * declared as input, and so on). Run-time exception is generated
     * otherwise.
     *
     * Objects in output vector may remain empty (like cv::Mat) --
     * G-API will automatically initialize output objects to proper formats.
     *
     * @note Don't construct GRunArgs/GRunArgsP objects manually, use
     * cv::gin()/cv::gout() wrappers instead.
     */
    void operator() (GRunArgs &&ins, GRunArgsP &&outs);          // Generic arg-to-arg
#if !defined(GAPI_STANDALONE)

    /**
     * @brief Execute an unary computation
     *
     * @overload
     * @param in input cv::Mat for unary computation
     * @param out output cv::Mat for unary computation
     * process.
     */
    void operator() (cv::Mat in, cv::Mat &out);                  // Unary overload

    /**
     * @brief Execute an unary computation
     *
     * @overload
     * @param in input cv::Mat for unary computation
     * @param out output cv::Scalar for unary computation
     * process.
     */
    void operator() (cv::Mat in, cv::Scalar &out);               // Unary overload (scalar)

    /**
     * @brief Execute a binary computation
     *
     * @overload
     * @param in1 first input cv::Mat for binary computation
     * @param in2 second input cv::Mat for binary computation
     * @param out output cv::Mat for binary computation
     * process.
     */
    void operator() (cv::Mat in1, cv::Mat in2, cv::Mat &out);    // Binary overload

    /**
     * @brief Execute an binary computation
     *
     * @overload
     * @param in1 first input cv::Mat for binary computation
     * @param in2 second input cv::Mat for binary computation
     * @param out output cv::Scalar for binary computation
     * process.
     */
    void operator() (cv::Mat in1, cv::Mat in2, cv::Scalar &out); // Binary overload (scalar)

    /**
     * @brief Execute a computation with arbitrary number of
     * inputs/outputs.
     *
     * @overload
     * @param ins vector of input cv::Mat objects to process by the
     * computation.
     * @param outs vector of output cv::Mat objects to produce by the
     * computation.
     *
     * Numbers of elements in ins/outs vectors must match numbers of
     * inputs/outputs which were used to define the source GComputation.
     */
    void operator() (const std::vector<cv::Mat> &ins,            // Compatibility overload
                     const std::vector<cv::Mat> &outs);
#endif  // !defined(GAPI_STANDALONE)
    /// @private
    Priv& priv();

    /**
     * @brief Check if compiled object is valid (non-empty)
     *
     * @return true if the object is runnable (valid), false otherwise
     */
    explicit operator bool () const;

    /**
     * @brief Vector of metadata this graph was compiled for.
     *
     * @return Unless _reshape_ is not supported, return value is the
     * same vector which was passed to cv::GComputation::compile() to
     * produce this compiled object. Otherwise, it is the latest
     * metadata vector passed to reshape() (if that call was
     * successful).
     */
    const GMetaArgs& metas() const; // Meta passed to compile()

    /**
     * @brief Vector of metadata descriptions of graph outputs
     *
     * @return vector with formats/resolutions of graph's output
     * objects, auto-inferred from input metadata vector by
     * operations which form this computation.
     *
     * @note GCompiled objects produced from the same
     * cv::GComputiation graph with different input metas may return
     * different values in this vector.
     */
    const GMetaArgs& outMetas() const;

    /**
     * @brief Check if the underlying backends support reshape or not.
     *
     * @return true if supported, false otherwise.
     */
    bool canReshape() const;

    /**
     * @brief Reshape a compiled graph to support new image
     * resolutions.
     *
     * Throws an exception if an error occurs.
     *
     * @param inMetas new metadata to reshape on. Vector size and
     * metadata shapes must match the computation's protocol.
     * @param args compilation arguments to use.
     */
    // FIXME: Why it requires compile args?
    void reshape(const GMetaArgs& inMetas, const GCompileArgs& args);

    /**
     * @brief Prepare inner kernels states for a new video-stream.
     *
     * GCompiled objects may be used to process video streams frame by frame.
     * In this case, a GCompiled is called on every image frame individually.
     * Starting OpenCV 4.4, some kernels in the graph may have their internal
     * states (see GAPI_OCV_KERNEL_ST for the OpenCV backend).
     * In this case, if user starts processing another video stream with
     * this GCompiled, this method needs to be called to let kernels re-initialize
     * their internal states to a new video stream.
     */
    void prepareForNewStream();

protected:
    /// @private
    std::shared_ptr<Priv> m_priv;
};
/** @} */

}

#endif // OPENCV_GAPI_GCOMPILED_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **represents**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **actually**: A class/struct defined in this file
- **GRunArgs**: A class/struct defined in this file

### Functions and Methods

- **call()**: A function/method defined in this file
- **OPENCV_GAPI_GCOMPILED_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/opencv_includes.hpp`
- `opencv2/gapi/own/assert.hpp`
- `vector`
- `opencv2/gapi/garg.hpp`

**Python Imports:**
- `the`
- `input`


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

