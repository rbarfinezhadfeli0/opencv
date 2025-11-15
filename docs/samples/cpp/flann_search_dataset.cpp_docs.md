# Documentation for `samples/cpp/flann_search_dataset.cpp`

## File Metadata

- **Full Path**: `samples/cpp/flann_search_dataset.cpp`
- **File Name**: `flann_search_dataset.cpp`
- **File Size**: 9,364 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/flann_search_dataset.cpp](../../samples/cpp/flann_search_dataset.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// flann_search_dataset.cpp
// Naive program to search a query picture in a dataset illustrating usage of FLANN

#include <iostream>
#include <vector>
#include "opencv2/core.hpp"
#include "opencv2/core/utils/filesystem.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/features2d.hpp"
#include "opencv2/flann.hpp"

using namespace cv;
using std::cout;
using std::endl;

#define _ORB_

const char* keys =
    "{ help h | | Print help message. }"
    "{ dataset | | Path to the images folder used as dataset. }"
    "{ image |   | Path to the image to search for in the dataset. }"
    "{ save |    | Path and filename where to save the flann structure to. }"
    "{ load |    | Path and filename where to load the flann structure from. }";

struct img_info {
    int img_index;
    unsigned int nbr_of_matches;

    img_info(int _img_index, unsigned int _nbr_of_matches)
        : img_index(_img_index)
        , nbr_of_matches(_nbr_of_matches)
    {}
};


int main( int argc, char* argv[] )
{
    //-- Test the program options
    CommandLineParser parser( argc, argv, keys );
    if (parser.has("help"))
    {
        parser.printMessage();
        return -1;
    }

    const cv::String img_path = parser.get<String>("image");
    Mat img = imread( samples::findFile( img_path ), IMREAD_GRAYSCALE );
    if (img.empty() )
    {
        cout << "Could not open the image "<< img_path << endl;
        return -1;
    }

    const cv::String db_path = parser.get<String>("dataset");
    if (!utils::fs::isDirectory(db_path))
    {
        cout << "Dataset folder "<< db_path.c_str() <<" doesn't exist!" << endl;
        return -1;
    }

    const cv::String load_db_path = parser.get<String>("load");
    if ((load_db_path != String()) && (!utils::fs::exists(load_db_path)))
    {
        cout << "File " << load_db_path.c_str()
             << " where to load the flann structure from doesn't exist!" << endl;
        return -1;
    }

    const cv::String save_db_path = parser.get<String>("save");

    //-- Step 1: Detect the keypoints using a detector, compute the descriptors
    //   in the folder containing the images of the dataset
#ifdef _SIFT_
    int minHessian = 400;
    Ptr<Feature2D> detector = SIFT::create( minHessian );
#elif defined(_ORB_)
    Ptr<Feature2D> detector = ORB::create();
#else
    cout << "Missing or unknown defined descriptor. "
            "Only SIFT and ORB are currently interfaced here" << endl;
    return -1;
#endif

    std::vector<KeyPoint> db_keypoints;
    Mat db_descriptors;
    std::vector<unsigned int> db_images_indice_range; //store the range of indices per image
    std::vector<int> db_indice_2_image_lut;           //match descriptor indice to its image

    db_images_indice_range.push_back(0);
    std::vector<cv::String> files;
    utils::fs::glob(db_path, cv::String(), files);
    for (std::vector<cv::String>::iterator itr = files.begin(); itr != files.end(); ++itr)
    {
        Mat tmp_img = imread( *itr, IMREAD_GRAYSCALE );
        if (!tmp_img.empty())
        {
            std::vector<KeyPoint> kpts;
            Mat descriptors;
            detector->detectAndCompute( tmp_img, noArray(), kpts, descriptors );

            db_keypoints.insert( db_keypoints.end(), kpts.begin(), kpts.end() );
            db_descriptors.push_back( descriptors );
            db_images_indice_range.push_back( db_images_indice_range.back()
                                              + static_cast<unsigned int>(kpts.size()) );
        }
    }

    //-- Set the LUT
    db_indice_2_image_lut.resize( db_images_indice_range.back() );
    const int nbr_of_imgs = static_cast<int>( db_images_indice_range.size()-1 );
    for (int i = 0; i < nbr_of_imgs; ++i)
    {
        const unsigned int first_indice = db_images_indice_range[i];
        const unsigned int last_indice = db_images_indice_range[i+1];
        std::fill( db_indice_2_image_lut.begin() + first_indice,
                   db_indice_2_image_lut.begin() + last_indice,
                   i );
    }

    //-- Step 2: build the structure storing the descriptors
#if defined(_SIFT_)
    cv::Ptr<flann::GenericIndex<cvflann::L2<float> > > index;
    if (load_db_path != String())
        index = cv::makePtr<flann::GenericIndex<cvflann::L2<float> > >(db_descriptors,
                                                             cvflann::SavedIndexParams(load_db_path));
    else
        index = cv::makePtr<flann::GenericIndex<cvflann::L2<float> > >(db_descriptors,
                                                             cvflann::KDTreeIndexParams(4));

#elif defined(_ORB_)
    cv::Ptr<flann::GenericIndex<cvflann::Hamming<unsigned char> > > index;
    if (load_db_path != String())
        index  = cv::makePtr<flann::GenericIndex<cvflann::Hamming<unsigned char> > >
                (db_descriptors, cvflann::SavedIndexParams(load_db_path));
    else
        index  = cv::makePtr<flann::GenericIndex<cvflann::Hamming<unsigned char> > >
                (db_descriptors, cvflann::LshIndexParams());
#else
    cout<< "Descriptor not listed. Set the proper FLANN distance for this descriptor" <<endl;
    return -1;
#endif
    if (save_db_path != String())
        index->save(save_db_path);


    // Return if no query image was set
    if (img_path == String())
        return 0;

    //-- Detect the keypoints and compute the descriptors for the query image
    std::vector<KeyPoint> img_keypoints;
    Mat img_descriptors;
    detector->detectAndCompute( img, noArray(), img_keypoints, img_descriptors );


    //-- Step 3: retrieve the descriptors in the dataset matching the ones of the query image
    // /!\ knnSearch doesn't follow OpenCV standards by not initialising empty Mat properties
    const int knn = 2;
    Mat indices(img_descriptors.rows, knn, CV_32S);
#if defined(_SIFT_)
#define DIST_TYPE float
    Mat dists(img_descriptors.rows, knn, CV_32F);
#elif defined(_ORB_)
#define DIST_TYPE int
    Mat dists(img_descriptors.rows, knn, CV_32S);
#endif
    index->knnSearch( img_descriptors, indices, dists, knn, cvflann::SearchParams(32) );

    //-- Filter matches using the Lowe's ratio test
    const float ratio_thresh = 0.7f;
    std::vector<DMatch> good_matches; //contains
    std::vector<unsigned int> matches_per_img_histogram( nbr_of_imgs, 0 );
    for (int i = 0; i < dists.rows; ++i)
    {
        if (dists.at<DIST_TYPE>(i,0) < ratio_thresh * dists.at<DIST_TYPE>(i,1))
        {
            const int indice_in_db = indices.at<int>(i,0);
            DMatch dmatch(i, indice_in_db, db_indice_2_image_lut[indice_in_db],
                          static_cast<float>(dists.at<DIST_TYPE>(i,0)));
            good_matches.push_back( dmatch );
            matches_per_img_histogram[ db_indice_2_image_lut[indice_in_db] ]++;
        }
    }


    //-- Step 4: find the dataset image with the highest proportion of matches
    std::multimap<float, img_info> images_infos;
    for (int i = 0; i < nbr_of_imgs; ++i)
    {
        const unsigned int nbr_of_matches = matches_per_img_histogram[i];
        if (nbr_of_matches < 4) //we need at leat 4 points for a homography
            continue;

        const unsigned int nbr_of_kpts = db_images_indice_range[i+1] - db_images_indice_range[i];
        const float inverse_proportion_of_retrieved_kpts =
                static_cast<float>(nbr_of_kpts) / static_cast<float>(nbr_of_matches);

        img_info info(i, nbr_of_matches);
        images_infos.insert( std::pair<float,img_info>(inverse_proportion_of_retrieved_kpts,
                                                       info) );
    }

    if (images_infos.begin() == images_infos.end())
    {
        cout<<"No good match could be found."<<endl;
        return 0;
    }

    //-- if there are several images with a similar proportion of matches,
    // select the one with the highest number of matches weighted by the
    // squared ratio of proportions
    const float best_matches_proportion = images_infos.begin()->first;
    float new_matches_proportion = best_matches_proportion;
    img_info best_img = images_infos.begin()->second;

    std::multimap<float, img_info>::iterator it = images_infos.begin();
    ++it;
    while ((it!=images_infos.end()) && (it->first < 1.1*best_matches_proportion))
    {
        const float ratio = new_matches_proportion / it->first;
        if( it->second.nbr_of_matches * (ratio * ratio) > best_img.nbr_of_matches)
        {
            new_matches_proportion = it->first;
            best_img = it->second;
        }
        ++it;
    }

    //-- Step 5: filter goodmatches that belong to the best image match of the dataset
    std::vector<DMatch> filtered_good_matches;
    for (std::vector<DMatch>::iterator itr(good_matches.begin()); itr != good_matches.end(); ++itr)
    {
        if (itr->imgIdx == best_img.img_index)
            filtered_good_matches.push_back(*itr);
    }

    //-- Retrieve the best image match from the dataset
    Mat db_img = imread( files[best_img.img_index], IMREAD_GRAYSCALE );

    //-- Draw matches
    Mat img_matches;
    drawMatches( img, img_keypoints, db_img, db_keypoints, filtered_good_matches, img_matches, Scalar::all(-1),
                 Scalar::all(-1), std::vector<char>(), DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS );

    //-- Show detected matches
    imshow("Good Matches", img_matches );
    waitKey();

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

### Classes and Structures

- **img_info**: A class/struct defined in this file

### Functions and Methods

- **_SIFT_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/utils/filesystem.hpp`
- `vector`
- `iostream`
- `opencv2/features2d.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`
- `opencv2/flann.hpp`

**Python Imports:**
- `doesn`
- `the`


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

