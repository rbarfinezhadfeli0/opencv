# Documentation for `docs/samples/dnn/face_detect.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/face_detect.cpp_docs.md`
- **File Name**: `face_detect.cpp_docs.md`
- **File Size**: 13,669 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/face_detect.cpp_docs.md](../../../docs/samples/dnn/face_detect.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/face_detect.cpp`

## File Metadata

- **Full Path**: `samples/dnn/face_detect.cpp`
- **File Name**: `face_detect.cpp`
- **File Size**: 10,705 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/dnn/face_detect.cpp](../../samples/dnn/face_detect.cpp)

## Purpose and Role

This file is located in the `samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/dnn.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/objdetect.hpp>

#include <iostream>

using namespace cv;
using namespace std;

static
void visualize(Mat& input, int frame, Mat& faces, double fps, int thickness = 2)
{
    std::string fpsString = cv::format("FPS : %.2f", (float)fps);
    if (frame >= 0)
        cout << "Frame " << frame << ", ";
    cout << "FPS: " << fpsString << endl;
    for (int i = 0; i < faces.rows; i++)
    {
        // Print results
        cout << "Face " << i
             << ", top-left coordinates: (" << faces.at<float>(i, 0) << ", " << faces.at<float>(i, 1) << "), "
             << "box width: " << faces.at<float>(i, 2)  << ", box height: " << faces.at<float>(i, 3) << ", "
             << "score: " << cv::format("%.2f", faces.at<float>(i, 14))
             << endl;

        // Draw bounding box
        rectangle(input, Rect2i(int(faces.at<float>(i, 0)), int(faces.at<float>(i, 1)), int(faces.at<float>(i, 2)), int(faces.at<float>(i, 3))), Scalar(0, 255, 0), thickness);
        // Draw landmarks
        circle(input, Point2i(int(faces.at<float>(i, 4)), int(faces.at<float>(i, 5))), 2, Scalar(255, 0, 0), thickness);
        circle(input, Point2i(int(faces.at<float>(i, 6)), int(faces.at<float>(i, 7))), 2, Scalar(0, 0, 255), thickness);
        circle(input, Point2i(int(faces.at<float>(i, 8)), int(faces.at<float>(i, 9))), 2, Scalar(0, 255, 0), thickness);
        circle(input, Point2i(int(faces.at<float>(i, 10)), int(faces.at<float>(i, 11))), 2, Scalar(255, 0, 255), thickness);
        circle(input, Point2i(int(faces.at<float>(i, 12)), int(faces.at<float>(i, 13))), 2, Scalar(0, 255, 255), thickness);
    }
    putText(input, fpsString, Point(0, 15), FONT_HERSHEY_SIMPLEX, 0.5, Scalar(0, 255, 0), 2);
}

int main(int argc, char** argv)
{
    CommandLineParser parser(argc, argv,
        "{help  h           |            | Print this message}"
        "{image1 i1         |            | Path to the input image1. Omit for detecting through VideoCapture}"
        "{image2 i2         |            | Path to the input image2. When image1 and image2 parameters given then the program try to find a face on both images and runs face recognition algorithm}"
        "{video v           | 0          | Path to the input video}"
        "{scale sc          | 1.0        | Scale factor used to resize input video frames}"
        "{fd_model fd       | face_detection_yunet_2021dec.onnx| Path to the model. Download yunet.onnx in https://github.com/opencv/opencv_zoo/tree/master/models/face_detection_yunet}"
        "{fr_model fr       | face_recognition_sface_2021dec.onnx | Path to the face recognition model. Download the model at https://github.com/opencv/opencv_zoo/tree/master/models/face_recognition_sface}"
        "{score_threshold   | 0.9        | Filter out faces of score < score_threshold}"
        "{nms_threshold     | 0.3        | Suppress bounding boxes of iou >= nms_threshold}"
        "{top_k             | 5000       | Keep top_k bounding boxes before NMS}"
        "{save s            | false      | Set true to save results. This flag is invalid when using camera}"
    );
    if (parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }

    String fd_modelPath = parser.get<String>("fd_model");
    String fr_modelPath = parser.get<String>("fr_model");

    float scoreThreshold = parser.get<float>("score_threshold");
    float nmsThreshold = parser.get<float>("nms_threshold");
    int topK = parser.get<int>("top_k");

    bool save = parser.get<bool>("save");
    float scale = parser.get<float>("scale");

    double cosine_similar_thresh = 0.363;
    double l2norm_similar_thresh = 1.128;

    //! [initialize_FaceDetectorYN]
    // Initialize FaceDetectorYN
    Ptr<FaceDetectorYN> detector = FaceDetectorYN::create(fd_modelPath, "", Size(320, 320), scoreThreshold, nmsThreshold, topK);
    //! [initialize_FaceDetectorYN]

    TickMeter tm;

    // If input is an image
    if (parser.has("image1"))
    {
        String input1 = parser.get<String>("image1");
        Mat image1 = imread(samples::findFile(input1));
        if (image1.empty())
        {
            std::cerr << "Cannot read image: " << input1 << std::endl;
            return 2;
        }

        int imageWidth = int(image1.cols * scale);
        int imageHeight = int(image1.rows * scale);
        resize(image1, image1, Size(imageWidth, imageHeight));
        tm.start();

        //! [inference]
        // Set input size before inference
        detector->setInputSize(image1.size());

        Mat faces1;
        detector->detect(image1, faces1);
        if (faces1.rows < 1)
        {
            std::cerr << "Cannot find a face in " << input1 << std::endl;
            return 1;
        }
        //! [inference]

        tm.stop();
        // Draw results on the input image
        visualize(image1, -1, faces1, tm.getFPS());

        // Save results if save is true
        if (save)
        {
            cout << "Saving result.jpg...\n";
            imwrite("result.jpg", image1);
        }

        // Visualize results
        imshow("image1", image1);
        pollKey();  // handle UI events to show content

        if (parser.has("image2"))
        {
            String input2 = parser.get<String>("image2");
            Mat image2 = imread(samples::findFile(input2));
            if (image2.empty())
            {
                std::cerr << "Cannot read image2: " << input2 << std::endl;
                return 2;
            }

            tm.reset();
            tm.start();
            detector->setInputSize(image2.size());

            Mat faces2;
            detector->detect(image2, faces2);
            if (faces2.rows < 1)
            {
                std::cerr << "Cannot find a face in " << input2 << std::endl;
                return 1;
            }
            tm.stop();
            visualize(image2, -1, faces2, tm.getFPS());
            if (save)
            {
                cout << "Saving result2.jpg...\n";
                imwrite("result2.jpg", image2);
            }
            imshow("image2", image2);
            pollKey();

            //! [initialize_FaceRecognizerSF]
            // Initialize FaceRecognizerSF
            Ptr<FaceRecognizerSF> faceRecognizer = FaceRecognizerSF::create(fr_modelPath, "");
            //! [initialize_FaceRecognizerSF]


            //! [facerecognizer]
            // Aligning and cropping facial image through the first face of faces detected.
            Mat aligned_face1, aligned_face2;
            faceRecognizer->alignCrop(image1, faces1.row(0), aligned_face1);
            faceRecognizer->alignCrop(image2, faces2.row(0), aligned_face2);

            // Run feature extraction with given aligned_face
            Mat feature1, feature2;
            faceRecognizer->feature(aligned_face1, feature1);
            feature1 = feature1.clone();
            faceRecognizer->feature(aligned_face2, feature2);
            feature2 = feature2.clone();
            //! [facerecognizer]

            //! [match]
            double cos_score = faceRecognizer->match(feature1, feature2, FaceRecognizerSF::DisType::FR_COSINE);
            double L2_score = faceRecognizer->match(feature1, feature2, FaceRecognizerSF::DisType::FR_NORM_L2);
            //! [match]

            if (cos_score >= cosine_similar_thresh)
            {
                std::cout << "They have the same identity;";
            }
            else
            {
                std::cout << "They have different identities;";
            }
            std::cout << " Cosine Similarity: " << cos_score << ", threshold: " << cosine_similar_thresh << ". (higher value means higher similarity, max 1.0)\n";

            if (L2_score <= l2norm_similar_thresh)
            {
                std::cout << "They have the same identity;";
            }
            else
            {
                std::cout << "They have different identities.";
            }
            std::cout << " NormL2 Distance: " << L2_score << ", threshold: " << l2norm_similar_thresh << ". (lower value means higher similarity, min 0.0)\n";
        }
        cout << "Press any key to exit..." << endl;
        waitKey(0);
    }
    else
    {
        int frameWidth, frameHeight;
        VideoCapture capture;
        std::string video = parser.get<string>("video");
        if (video.size() == 1 && isdigit(video[0]))
            capture.open(parser.get<int>("video"));
        else
            capture.open(samples::findFileOrKeep(video));  // keep GStreamer pipelines
        if (capture.isOpened())
        {
            frameWidth = int(capture.get(CAP_PROP_FRAME_WIDTH) * scale);
            frameHeight = int(capture.get(CAP_PROP_FRAME_HEIGHT) * scale);
            cout << "Video " << video
                << ": width=" << frameWidth
                << ", height=" << frameHeight
                << endl;
        }
        else
        {
            cout << "Could not initialize video capturing: " << video << "\n";
            return 1;
        }

        detector->setInputSize(Size(frameWidth, frameHeight));

        cout << "Press 'SPACE' to save frame, any other key to exit..." << endl;
        int nFrame = 0;
        for (;;)
        {
            // Get frame
            Mat frame;
            if (!capture.read(frame))
            {
                cerr << "Can't grab frame! Stop\n";
                break;
            }

            resize(frame, frame, Size(frameWidth, frameHeight));

            // Inference
            Mat faces;
            tm.start();
            detector->detect(frame, faces);
            tm.stop();

            Mat result = frame.clone();
            // Draw results on the input image
            visualize(result, nFrame, faces, tm.getFPS());

            // Visualize results
            imshow("Live", result);

            int key = waitKey(1);
            bool saveFrame = save;
            if (key == ' ')
            {
                saveFrame = true;
                key = 0;  // handled
            }

            if (saveFrame)
            {
                std::string frame_name = cv::format("frame_%05d.png", nFrame);
                std::string result_name = cv::format("result_%05d.jpg", nFrame);
                cout << "Saving '" << frame_name << "' and '" << result_name << "' ...\n";
                imwrite(frame_name, frame);
                imwrite(result_name, result);
            }

            ++nFrame;

            if (key > 0)
                break;
        }
        cout << "Processed " << nFrame << " frames" << endl;
    }
    cout << "Done." << endl;
    return 0;
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
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/highgui.hpp`
- `opencv2/objdetect.hpp`
- `opencv2/dnn.hpp`


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

