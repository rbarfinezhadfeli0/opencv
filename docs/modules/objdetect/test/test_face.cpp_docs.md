# Documentation for `modules/objdetect/test/test_face.cpp`

## File Metadata

- **Full Path**: `modules/objdetect/test/test_face.cpp`
- **File Name**: `test_face.cpp`
- **File Size**: 7,611 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/objdetect/test/test_face.cpp](../../../modules/objdetect/test/test_face.cpp)

## Purpose and Role

This file is located in the `modules/objdetect/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"

namespace opencv_test { namespace {

// label format:
//   image_name
//   num_face
//   face_1
//   face_..
//   face_num
std::map<std::string, Mat> blobFromTXT(const std::string& path, int numCoords)
{
    std::ifstream ifs(path.c_str());
    CV_Assert(ifs.is_open());

    std::map<std::string, Mat> gt;

    Mat faces;
    int faceNum = -1;
    int faceCount = 0;
    for (std::string line, key; getline(ifs, line); )
    {
        std::istringstream iss(line);
        if (line.find(".png") != std::string::npos)
        {
            // Get filename
            iss >> key;
        }
        else if (line.find(" ") == std::string::npos)
        {
            // Get the number of faces
            iss >> faceNum;
        }
        else
        {
            // Get faces
            Mat face(1, numCoords, CV_32FC1);
            for (int j = 0; j < numCoords; j++)
            {
                iss >> face.at<float>(0, j);
            }
            faces.push_back(face);
            faceCount++;
        }

        if (faceCount == faceNum)
        {
            // Store faces
            gt[key] = faces;

            faces.release();
            faceNum = -1;
            faceCount = 0;
        }
    }

    return gt;
}

TEST(Objdetect_face_detection, regression)
{
    // Pre-set params
    float scoreThreshold = 0.7f;
    float matchThreshold = 0.7f;
    float l2disThreshold = 15.0f;
    int numLM = 5;
    int numCoords = 4 + 2 * numLM;

    // Load ground truth labels
    std::map<std::string, Mat> gt = blobFromTXT(findDataFile("dnn_face/detection/cascades_labels.txt"), numCoords);

    // Initialize detector
    std::string model = findDataFile("dnn/onnx/models/yunet-202303.onnx", false);
    Ptr<FaceDetectorYN> faceDetector = FaceDetectorYN::create(model, "", Size(300, 300));
    faceDetector->setScoreThreshold(0.7f);

    // Detect and match
    for (auto item: gt)
    {
        std::string imagePath = findDataFile("cascadeandhog/images/" + item.first);
        Mat image = imread(imagePath);

        // Set input size
        faceDetector->setInputSize(image.size());

        // Run detection
        Mat faces;
        faceDetector->detect(image, faces);
        // std::cout << item.first << " " << item.second.rows << " " << faces.rows << std::endl;

        // Match bboxes and landmarks
        std::vector<bool> matchedItem(item.second.rows, false);
        for (int i = 0; i < faces.rows; i++)
        {
            if (faces.at<float>(i, numCoords) < scoreThreshold)
                continue;

            bool boxMatched = false;
            std::vector<bool> lmMatched(numLM, false);
            cv::Rect2f resBox(faces.at<float>(i, 0), faces.at<float>(i, 1), faces.at<float>(i, 2), faces.at<float>(i, 3));
            for (int j = 0; j < item.second.rows && !boxMatched; j++)
            {
                if (matchedItem[j])
                    continue;

                // Retrieve bbox and compare IoU
                cv::Rect2f gtBox(item.second.at<float>(j, 0), item.second.at<float>(j, 1), item.second.at<float>(j, 2), item.second.at<float>(j, 3));
                double interArea = (resBox & gtBox).area();
                double iou = interArea / (resBox.area() + gtBox.area() - interArea);
                if (iou >= matchThreshold)
                {
                    boxMatched = true;
                    matchedItem[j] = true;
                }

                // Match landmarks if bbox is matched
                if (!boxMatched)
                    continue;
                for (int lmIdx = 0; lmIdx < numLM; lmIdx++)
                {
                    float gtX = item.second.at<float>(j, 4 + 2 * lmIdx);
                    float gtY = item.second.at<float>(j, 4 + 2 * lmIdx + 1);
                    float resX = faces.at<float>(i, 4 + 2 * lmIdx);
                    float resY = faces.at<float>(i, 4 + 2 * lmIdx + 1);
                    float l2dis = cv::sqrt((gtX - resX) * (gtX - resX) + (gtY - resY) * (gtY - resY));

                    if (l2dis <= l2disThreshold)
                    {
                        lmMatched[lmIdx] = true;
                    }
                }
                break;
            }
            EXPECT_TRUE(boxMatched) << "In image " << item.first << ", cannot match resBox " << resBox << " with any ground truth.";
            if (boxMatched)
            {
                EXPECT_TRUE(std::all_of(lmMatched.begin(), lmMatched.end(), [](bool v) { return v; })) << "In image " << item.first << ", resBox " << resBox << " matched but its landmarks failed to match.";
            }
        }
    }
}

TEST(Objdetect_face_recognition, regression)
{
    // Pre-set params
    float score_thresh = 0.9f;
    float nms_thresh = 0.3f;
    double cosine_similar_thresh = 0.363;
    double l2norm_similar_thresh = 1.128;

    // Load ground truth labels
    std::ifstream ifs(findDataFile("dnn_face/recognition/cascades_label.txt").c_str());
    CV_Assert(ifs.is_open());

    std::set<std::string> fSet;
    std::map<std::string, Mat> featureMap;
    std::map<std::pair<std::string, std::string>, int> gtMap;


    for (std::string line, key; getline(ifs, line);)
    {
        std::string fname1, fname2;
        int label;
        std::istringstream iss(line);
        iss>>fname1>>fname2>>label;
        // std::cout<<fname1<<" "<<fname2<<" "<<label<<std::endl;

        fSet.insert(fname1);
        fSet.insert(fname2);
        gtMap[std::make_pair(fname1, fname2)] = label;
    }

    // Initialize detector
    std::string detect_model = findDataFile("dnn/onnx/models/yunet-202303.onnx", false);
    Ptr<FaceDetectorYN> faceDetector = FaceDetectorYN::create(detect_model, "", Size(150, 150), score_thresh, nms_thresh);

    std::string recog_model = findDataFile("dnn/onnx/models/face_recognizer_fast.onnx", false);
    Ptr<FaceRecognizerSF> faceRecognizer = FaceRecognizerSF::create(recog_model, "");

    // Detect and match
    for (auto fname: fSet)
    {
        std::string imagePath = findDataFile("dnn_face/recognition/" + fname);
        Mat image = imread(imagePath);

        Mat faces;
        faceDetector->detect(image, faces);

        Mat aligned_face;
        faceRecognizer->alignCrop(image, faces.row(0), aligned_face);

        Mat feature;
        faceRecognizer->feature(aligned_face, feature);

        featureMap[fname] = feature.clone();
    }

    for (auto item: gtMap)
    {
        Mat feature1 = featureMap[item.first.first];
        Mat feature2 = featureMap[item.first.second];
        int label = item.second;

        double cos_score = faceRecognizer->match(feature1, feature2, FaceRecognizerSF::DisType::FR_COSINE);
        double L2_score = faceRecognizer->match(feature1, feature2, FaceRecognizerSF::DisType::FR_NORM_L2);

        EXPECT_TRUE(label == 0 ? cos_score <= cosine_similar_thresh : cos_score > cosine_similar_thresh) << "Cosine match result of images " << item.first.first << " and " << item.first.second << " is different from ground truth (score: "<< cos_score <<";Thresh: "<< cosine_similar_thresh <<").";
        EXPECT_TRUE(label == 0 ? L2_score > l2norm_similar_thresh : L2_score <= l2norm_similar_thresh) << "L2norm match result of images " << item.first.first << " and " << item.first.second << " is different from ground truth (score: "<< L2_score <<";Thresh: "<< l2norm_similar_thresh <<").";
    }
}

}} // namespace
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
- `test_precomp.hpp`

**Python Imports:**
- `ground`


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

