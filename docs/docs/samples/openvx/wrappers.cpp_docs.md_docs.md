# Documentation for `docs/samples/openvx/wrappers.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/openvx/wrappers.cpp_docs.md`
- **File Name**: `wrappers.cpp_docs.md`
- **File Size**: 9,671 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/openvx/wrappers.cpp_docs.md](../../../docs/samples/openvx/wrappers.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/openvx` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/openvx/wrappers.cpp`

## File Metadata

- **Full Path**: `samples/openvx/wrappers.cpp`
- **File Name**: `wrappers.cpp`
- **File Size**: 6,540 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/openvx/wrappers.cpp](../../samples/openvx/wrappers.cpp)

## Purpose and Role

This file is located in the `samples/openvx` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <stdexcept>

//wrappers
#include "ivx.hpp"

//OpenCV includes
#include "opencv2/core.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"

enum UserMemoryMode
{
    COPY, USER_MEM, MAP
};

ivx::Graph createProcessingGraph(ivx::Image& inputImage, ivx::Image& outputImage);
int ovxDemo(std::string inputPath, UserMemoryMode mode);


ivx::Graph createProcessingGraph(ivx::Image& inputImage, ivx::Image& outputImage)
{
    using namespace ivx;

    Context context = inputImage.get<Context>();
    Graph graph = Graph::create(context);

    vx_uint32 width  = inputImage.width();
    vx_uint32 height = inputImage.height();

    // Intermediate images
    Image
        smoothed  = Image::createVirtual(graph),
        cannied   = Image::createVirtual(graph),
        halfImg   = Image::create(context, width, height, VX_DF_IMAGE_U8),
        halfCanny = Image::create(context, width, height, VX_DF_IMAGE_U8);

    // Constants
    vx_uint32 threshCannyMin = 127;
    vx_uint32 threshCannyMax = 192;
    Threshold threshCanny = Threshold::createRange(context, VX_TYPE_UINT8, threshCannyMin, threshCannyMax);

    ivx::Scalar alpha = ivx::Scalar::create<VX_TYPE_FLOAT32>(context, 0.5);

    // Sequence of some image operations
    // Node can also be added in function-like style
    nodes::gaussian3x3(graph, inputImage, smoothed);
    Node::create(graph, VX_KERNEL_CANNY_EDGE_DETECTOR, smoothed, threshCanny,
                 ivx::Scalar::create<VX_TYPE_INT32>(context, 3),
                 ivx::Scalar::create<VX_TYPE_ENUM>(context, VX_NORM_L2), cannied);
    Node::create(graph, VX_KERNEL_ACCUMULATE_WEIGHTED, inputImage, alpha, halfImg);
    Node::create(graph, VX_KERNEL_ACCUMULATE_WEIGHTED, cannied, alpha, halfCanny);
    Node::create(graph, VX_KERNEL_ADD, halfImg, halfCanny,
                 ivx::Scalar::create<VX_TYPE_ENUM>(context, VX_CONVERT_POLICY_SATURATE), outputImage);

    graph.verify();

    return graph;
}


int ovxDemo(std::string inputPath, UserMemoryMode mode)
{
    using namespace cv;
    using namespace ivx;

    Mat image = imread(inputPath, IMREAD_GRAYSCALE);
    if (image.empty()) return -1;

    //check image format
    if (image.depth() != CV_8U || image.channels() != 1) return -1;

    try
    {
        Context context = Context::create();
        //put user data from cv::Mat to vx_image
        vx_df_image color = Image::matTypeToFormat(image.type());
        vx_uint32 width = image.cols, height = image.rows;
        Image ivxImage;
        if (mode == COPY)
        {
            ivxImage = Image::create(context, width, height, color);
            ivxImage.copyFrom(0, image);
        }
        else
        {
            ivxImage = Image::createFromHandle(context, color, Image::createAddressing(image), image.data);
        }

        Image ivxResult;
        Image::Patch resultPatch;
        Mat output;
        if (mode == COPY || mode == MAP)
        {
            //we will copy or map data from vx_image to cv::Mat
            ivxResult = ivx::Image::create(context, width, height, VX_DF_IMAGE_U8);
        }
        else // if (mode == MAP_TO_VX)
        {
            //create vx_image based on user data, no copying required
            output = cv::Mat(height, width, CV_8U, cv::Scalar(0));
            ivxResult = Image::createFromHandle(context, Image::matTypeToFormat(CV_8U),
                                                Image::createAddressing(output), output.data);
        }

        Graph graph = createProcessingGraph(ivxImage, ivxResult);

        // Graph execution
        graph.process();

        //getting resulting image in cv::Mat
        if (mode == COPY)
        {
            ivxResult.copyTo(0, output);
        }
        else if (mode == MAP)
        {
            //create cv::Mat based on vx_image mapped data
            resultPatch.map(ivxResult, 0, ivxResult.getValidRegion());
            //generally this is very bad idea!
            //but in our case unmap() won't happen until output is in use
            output = resultPatch.getMat();
        }
        else // if (mode == MAP_TO_VX)
        {
#ifdef VX_VERSION_1_1
            //we should take user memory back from vx_image before using it (even before reading)
            ivxResult.swapHandle();
#endif
        }

        //here output goes
        cv::imshow("processing result", output);
        cv::waitKey(0);

        cv::destroyAllWindows();

#ifdef VX_VERSION_1_1
        if (mode != COPY)
        {
            //we should take user memory back before release
            //(it's not done automatically according to standard)
            ivxImage.swapHandle();
            if (mode == USER_MEM) ivxResult.swapHandle();
        }
#endif

        //the line is unnecessary since unmapping is done on destruction of patch
        //resultPatch.unmap();
    }
    catch (const ivx::RuntimeError& e)
    {
        std::cerr << "Error: code = " << e.status() << ", message = " << e.what() << std::endl;
        return e.status();
    }
    catch (const ivx::WrapperError& e)
    {
        std::cerr << "Error: message = " << e.what() << std::endl;
        return -1;
    }

    return 0;
}


int main(int argc, char *argv[])
{
    const std::string keys =
        "{help h usage ? | | }"
        "{image    | <none> | image to be processed}"
        "{mode | copy | user memory interaction mode: \n"
        "copy: create VX images and copy data to/from them\n"
        "user_mem: use handles to user-allocated memory\n"
        "map: map resulting VX image to user memory}"
        ;

    cv::CommandLineParser parser(argc, argv, keys);
    parser.about("OpenVX interoperability sample demonstrating OpenVX wrappers usage."
                 "The application loads an image, processes it with OpenVX graph and outputs result in a window");
    if (parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }
    std::string imgPath = parser.get<std::string>("image");
    std::string modeString = parser.get<std::string>("mode");
    UserMemoryMode mode;
    if(modeString == "copy")
    {
        mode = COPY;
    }
    else if(modeString == "user_mem")
    {
        mode = USER_MEM;
    }
    else if(modeString == "map")
    {
        mode = MAP;
    }
    else
    {
        std::cerr << modeString << ": unknown memory mode" << std::endl;
        return -1;
    }

    if (!parser.check())
    {
        parser.printErrors();
        return -1;
    }

    return ovxDemo(imgPath, mode);
}
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **VX_VERSION_1_1()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/imgcodecs.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `stdexcept`
- `opencv2/core.hpp`
- `ivx.hpp`
- `opencv2/highgui.hpp`

**Python Imports:**
- `cv`
- `them`
- `vx_image`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

