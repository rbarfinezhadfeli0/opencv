# Documentation for `docs/samples/dnn/scene_text_spotting.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/scene_text_spotting.cpp_docs.md`
- **File Name**: `scene_text_spotting.cpp_docs.md`
- **File Size**: 9,331 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/scene_text_spotting.cpp_docs.md](../../../docs/samples/dnn/scene_text_spotting.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/scene_text_spotting.cpp`

## File Metadata

- **Full Path**: `samples/dnn/scene_text_spotting.cpp`
- **File Name**: `scene_text_spotting.cpp`
- **File Size**: 6,311 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/dnn/scene_text_spotting.cpp](../../samples/dnn/scene_text_spotting.cpp)

## Purpose and Role

This file is located in the `samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <fstream>

#include <opencv2/imgproc.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/dnn/dnn.hpp>

using namespace cv;
using namespace cv::dnn;

std::string keys =
        "{ help  h                          | | Print help message. }"
        "{ inputImage i                     | | Path to an input image. Skip this argument to capture frames from a camera. }"
        "{ detModelPath dmp                 | | Path to a binary .onnx model for detection. "
            "Download links are provided in doc/tutorials/dnn/dnn_text_spotting/dnn_text_spotting.markdown}"
        "{ recModelPath rmp                 | | Path to a binary .onnx model for recognition. "
            "Download links are provided in doc/tutorials/dnn/dnn_text_spotting/dnn_text_spotting.markdown}"
        "{ inputHeight ih                   |736| image height of the model input. It should be multiple by 32.}"
        "{ inputWidth iw                    |736| image width of the model input. It should be multiple by 32.}"
        "{ RGBInput rgb                     |0| 0: imread with flags=IMREAD_GRAYSCALE; 1: imread with flags=IMREAD_COLOR. }"
        "{ binaryThreshold bt               |0.3| Confidence threshold of the binary map. }"
        "{ polygonThreshold pt              |0.5| Confidence threshold of polygons. }"
        "{ maxCandidate max                 |200| Max candidates of polygons. }"
        "{ unclipRatio ratio                |2.0| unclip ratio. }"
        "{ vocabularyPath vp                | alphabet_36.txt | Path to benchmarks for evaluation. "
            "Download links are provided in doc/tutorials/dnn/dnn_text_spotting/dnn_text_spotting.markdown}";

void fourPointsTransform(const Mat& frame, const Point2f vertices[], Mat& result);
bool sortPts(const Point& p1, const Point& p2);

int main(int argc, char** argv)
{
    // Parse arguments
    CommandLineParser parser(argc, argv, keys);
    parser.about("Use this script to run an end-to-end inference sample of textDetectionModel and textRecognitionModel APIs\n"
                 "Use -h for more information");
    if (argc == 1 || parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }

    float binThresh = parser.get<float>("binaryThreshold");
    float polyThresh = parser.get<float>("polygonThreshold");
    uint maxCandidates = parser.get<uint>("maxCandidate");
    String detModelPath = parser.get<String>("detModelPath");
    String recModelPath = parser.get<String>("recModelPath");
    String vocPath = parser.get<String>("vocabularyPath");
    double unclipRatio = parser.get<double>("unclipRatio");
    int height = parser.get<int>("inputHeight");
    int width = parser.get<int>("inputWidth");
    int imreadRGB = parser.get<int>("RGBInput");

    if (!parser.check())
    {
        parser.printErrors();
        return 1;
    }

    // Load networks
    CV_Assert(!detModelPath.empty());
    TextDetectionModel_DB detector(detModelPath);
    detector.setBinaryThreshold(binThresh)
            .setPolygonThreshold(polyThresh)
            .setUnclipRatio(unclipRatio)
            .setMaxCandidates(maxCandidates);

    CV_Assert(!recModelPath.empty());
    TextRecognitionModel recognizer(recModelPath);

    // Load vocabulary
    CV_Assert(!vocPath.empty());
    std::ifstream vocFile;
    vocFile.open(samples::findFile(vocPath));
    CV_Assert(vocFile.is_open());
    String vocLine;
    std::vector<String> vocabulary;
    while (std::getline(vocFile, vocLine)) {
        vocabulary.push_back(vocLine);
    }
    recognizer.setVocabulary(vocabulary);
    recognizer.setDecodeType("CTC-greedy");

    // Parameters for Detection
    double detScale = 1.0 / 255.0;
    Size detInputSize = Size(width, height);
    Scalar detMean = Scalar(122.67891434, 116.66876762, 104.00698793);
    detector.setInputParams(detScale, detInputSize, detMean);

    // Parameters for Recognition
    double recScale = 1.0 / 127.5;
    Scalar recMean = Scalar(127.5);
    Size recInputSize = Size(100, 32);
    recognizer.setInputParams(recScale, recInputSize, recMean);

    // Create a window
    static const std::string winName = "Text_Spotting";

    // Input data
    Mat frame = imread(samples::findFile(parser.get<String>("inputImage")));
    std::cout << frame.size << std::endl;

    // Inference
    std::vector< std::vector<Point> > detResults;
    detector.detect(frame, detResults);
    Mat frame2 = frame.clone();

    if (detResults.size() > 0) {
        // Text Recognition
        Mat recInput;
        if (!imreadRGB) {
            cvtColor(frame, recInput, cv::COLOR_BGR2GRAY);
        } else {
            recInput = frame;
        }
        std::vector< std::vector<Point> > contours;
        for (uint i = 0; i < detResults.size(); i++)
        {
            const auto& quadrangle = detResults[i];
            CV_CheckEQ(quadrangle.size(), (size_t)4, "");

            contours.emplace_back(quadrangle);

            std::vector<Point2f> quadrangle_2f;
            for (int j = 0; j < 4; j++)
                quadrangle_2f.emplace_back(quadrangle[j]);

            // Transform and Crop
            Mat cropped;
            fourPointsTransform(recInput, &quadrangle_2f[0], cropped);

            std::string recognitionResult = recognizer.recognize(cropped);
            std::cout << i << ": '" << recognitionResult << "'" << std::endl;

            putText(frame2, recognitionResult, quadrangle[3], FONT_HERSHEY_SIMPLEX, 1, Scalar(0, 0, 255), 2);
        }
        polylines(frame2, contours, true, Scalar(0, 255, 0), 2);
    } else {
        std::cout << "No Text Detected." << std::endl;
    }
    imshow(winName, frame2);
    waitKey();

    return 0;
}

void fourPointsTransform(const Mat& frame, const Point2f vertices[], Mat& result)
{
    const Size outputSize = Size(100, 32);

    Point2f targetVertices[4] = {
        Point(0, outputSize.height - 1),
        Point(0, 0),
        Point(outputSize.width - 1, 0),
        Point(outputSize.width - 1, outputSize.height - 1)
    };
    Mat rotationMatrix = getPerspectiveTransform(vertices, targetVertices);

    warpPerspective(frame, result, rotationMatrix, outputSize);

#if 0
    imshow("roi", result);
    waitKey();
#endif
}

bool sortPts(const Point& p1, const Point& p2)
{
    return p1.x < p2.x;
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `fstream`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/dnn/dnn.hpp`
- `opencv2/highgui.hpp`

**Python Imports:**
- `a`


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

