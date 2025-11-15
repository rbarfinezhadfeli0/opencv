# Documentation for `docs/samples/gpu/stereo_match.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/gpu/stereo_match.cpp_docs.md`
- **File Name**: `stereo_match.cpp_docs.md`
- **File Size**: 13,792 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/gpu/stereo_match.cpp_docs.md](../../../docs/samples/gpu/stereo_match.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/gpu/stereo_match.cpp`

## File Metadata

- **Full Path**: `samples/gpu/stereo_match.cpp`
- **File Name**: `stereo_match.cpp`
- **File Size**: 10,638 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/gpu/stereo_match.cpp](../../samples/gpu/stereo_match.cpp)

## Purpose and Role

This file is located in the `samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <string>
#include <sstream>
#include <iomanip>
#include <stdexcept>
#include <opencv2/core/utility.hpp>
#include "opencv2/cudastereo.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"

using namespace cv;
using namespace std;

bool help_showed = false;

struct Params
{
    Params();
    static Params read(int argc, char** argv);

    string left;
    string right;

    string method_str() const
    {
        switch (method)
        {
        case BM: return "BM";
        case BP: return "BP";
        case CSBP: return "CSBP";
        }
        return "";
    }
    enum {BM, BP, CSBP} method;
    int ndisp; // Max disparity + 1
};


struct App
{
    App(const Params& p);
    void run();
    void handleKey(char key);
    void printParams() const;

    void workBegin() { work_begin = getTickCount(); }
    void workEnd()
    {
        int64 d = getTickCount() - work_begin;
        double f = getTickFrequency();
        work_fps = f / d;
    }

    string text() const
    {
        stringstream ss;
        ss << "(" << p.method_str() << ") FPS: " << setiosflags(ios::left)
            << setprecision(4) << work_fps;
        return ss.str();
    }
private:
    Params p;
    bool running;

    Mat left_src, right_src;
    Mat left, right;
    cuda::GpuMat d_left, d_right;

    Ptr<cuda::StereoBM> bm;
    Ptr<cuda::StereoBeliefPropagation> bp;
    Ptr<cuda::StereoConstantSpaceBP> csbp;

    int64 work_begin;
    double work_fps;
};

static void printHelp()
{
    cout << "Usage: stereo_match\n"
        << "\t--left <left_view> --right <right_view> # must be rectified\n"
        << "\t--method <stereo_match_method> # BM | BP | CSBP\n"
        << "\t--ndisp <number> # number of disparity levels\n";
    help_showed = true;
}

int main(int argc, char** argv)
{
    try
    {
        if (argc < 2)
        {
            printHelp();
            return 1;
        }
        Params args = Params::read(argc, argv);
        if (help_showed)
            return -1;
        App app(args);
        app.run();
    }
    catch (const exception& e)
    {
        cout << "error: " << e.what() << endl;
    }
    return 0;
}


Params::Params()
{
    method = BM;
    ndisp = 64;
}


Params Params::read(int argc, char** argv)
{
    Params p;

    for (int i = 1; i < argc; i++)
    {
        if (string(argv[i]) == "--left") p.left = argv[++i];
        else if (string(argv[i]) == "--right") p.right = argv[++i];
        else if (string(argv[i]) == "--method")
        {
            if (string(argv[i + 1]) == "BM") p.method = BM;
            else if (string(argv[i + 1]) == "BP") p.method = BP;
            else if (string(argv[i + 1]) == "CSBP") p.method = CSBP;
            else throw runtime_error("unknown stereo match method: " + string(argv[i + 1]));
            i++;
        }
        else if (string(argv[i]) == "--ndisp") p.ndisp = atoi(argv[++i]);
        else if (string(argv[i]) == "--help") printHelp();
        else throw runtime_error("unknown key: " + string(argv[i]));
    }

    return p;
}


App::App(const Params& params)
    : p(params), running(false)
{
    cv::cuda::printShortCudaDeviceInfo(cv::cuda::getDevice());

    cout << "stereo_match_gpu sample\n";
    cout << "\nControls:\n"
        << "\tesc - exit\n"
        << "\tp - print current parameters\n"
        << "\tg - convert source images into gray\n"
        << "\tm - change stereo match method\n"
        << "\ts - change Sobel prefiltering flag (for BM only)\n"
        << "\t1/q - increase/decrease maximum disparity\n"
        << "\t2/w - increase/decrease window size (for BM only)\n"
        << "\t3/e - increase/decrease iteration count (for BP and CSBP only)\n"
        << "\t4/r - increase/decrease level count (for BP and CSBP only)\n";
}


void App::run()
{
    // Load images
    left_src = imread(p.left);
    right_src = imread(p.right);
    if (left_src.empty()) throw runtime_error("can't open file \"" + p.left + "\"");
    if (right_src.empty()) throw runtime_error("can't open file \"" + p.right + "\"");
    cvtColor(left_src, left, COLOR_BGR2GRAY);
    cvtColor(right_src, right, COLOR_BGR2GRAY);
    d_left.upload(left);
    d_right.upload(right);

    imshow("left", left);
    imshow("right", right);

    // Set common parameters
    bm = cuda::createStereoBM(p.ndisp);
    bp = cuda::createStereoBeliefPropagation(p.ndisp);
    csbp = cv::cuda::createStereoConstantSpaceBP(p.ndisp);

    // Prepare disparity map of specified type
    Mat disp(left.size(), CV_8U);
    cuda::GpuMat d_disp(left.size(), CV_8U);

    cout << endl;
    printParams();

    running = true;
    while (running)
    {
        workBegin();
        switch (p.method)
        {
        case Params::BM:
            if (d_left.channels() > 1 || d_right.channels() > 1)
            {
                cout << "BM doesn't support color images\n";
                cvtColor(left_src, left, COLOR_BGR2GRAY);
                cvtColor(right_src, right, COLOR_BGR2GRAY);
                cout << "image_channels: " << left.channels() << endl;
                d_left.upload(left);
                d_right.upload(right);
                imshow("left", left);
                imshow("right", right);
            }
            bm->compute(d_left, d_right, d_disp);
            break;
        case Params::BP: bp->compute(d_left, d_right, d_disp); break;
        case Params::CSBP: csbp->compute(d_left, d_right, d_disp); break;
        }
        workEnd();

        // Show results
        d_disp.download(disp);
        putText(disp, text(), Point(5, 25), FONT_HERSHEY_SIMPLEX, 1.0, Scalar::all(255));
        imshow("disparity", (Mat_<uchar>)disp);

        handleKey((char)waitKey(3));
    }
}


void App::printParams() const
{
    cout << "--- Parameters ---\n";
    cout << "image_size: (" << left.cols << ", " << left.rows << ")\n";
    cout << "image_channels: " << left.channels() << endl;
    cout << "method: " << p.method_str() << endl
        << "ndisp: " << p.ndisp << endl;
    switch (p.method)
    {
    case Params::BM:
        cout << "win_size: " << bm->getBlockSize() << endl;
        cout << "prefilter_sobel: " << bm->getPreFilterType() << endl;
        break;
    case Params::BP:
        cout << "iter_count: " << bp->getNumIters() << endl;
        cout << "level_count: " << bp->getNumLevels() << endl;
        break;
    case Params::CSBP:
        cout << "iter_count: " << csbp->getNumIters() << endl;
        cout << "level_count: " << csbp->getNumLevels() << endl;
        break;
    }
    cout << endl;
}


void App::handleKey(char key)
{
    switch (key)
    {
    case 27:
        running = false;
        break;
    case 'p': case 'P':
        printParams();
        break;
    case 'g': case 'G':
        if (left.channels() == 1 && p.method != Params::BM)
        {
            left = left_src;
            right = right_src;
        }
        else
        {
            cvtColor(left_src, left, COLOR_BGR2GRAY);
            cvtColor(right_src, right, COLOR_BGR2GRAY);
        }
        d_left.upload(left);
        d_right.upload(right);
        cout << "image_channels: " << left.channels() << endl;
        imshow("left", left);
        imshow("right", right);
        break;
    case 'm': case 'M':
        switch (p.method)
        {
        case Params::BM:
            p.method = Params::BP;
            break;
        case Params::BP:
            p.method = Params::CSBP;
            break;
        case Params::CSBP:
            p.method = Params::BM;
            break;
        }
        cout << "method: " << p.method_str() << endl;
        break;
    case 's': case 'S':
        if (p.method == Params::BM)
        {
            switch (bm->getPreFilterType())
            {
            case 0:
                bm->setPreFilterType(cv::StereoBM::PREFILTER_XSOBEL);
                break;
            case cv::StereoBM::PREFILTER_XSOBEL:
                bm->setPreFilterType(0);
                break;
            }
            cout << "prefilter_sobel: " << bm->getPreFilterType() << endl;
        }
        break;
    case '1':
        p.ndisp = p.ndisp == 1 ? 8 : p.ndisp + 8;
        cout << "ndisp: " << p.ndisp << endl;
        bm->setNumDisparities(p.ndisp);
        bp->setNumDisparities(p.ndisp);
        csbp->setNumDisparities(p.ndisp);
        break;
    case 'q': case 'Q':
        p.ndisp = max(p.ndisp - 8, 1);
        cout << "ndisp: " << p.ndisp << endl;
        bm->setNumDisparities(p.ndisp);
        bp->setNumDisparities(p.ndisp);
        csbp->setNumDisparities(p.ndisp);
        break;
    case '2':
        if (p.method == Params::BM)
        {
            bm->setBlockSize(min(bm->getBlockSize() + 1, 51));
            cout << "win_size: " << bm->getBlockSize() << endl;
        }
        break;
    case 'w': case 'W':
        if (p.method == Params::BM)
        {
            bm->setBlockSize(max(bm->getBlockSize() - 1, 2));
            cout << "win_size: " << bm->getBlockSize() << endl;
        }
        break;
    case '3':
        if (p.method == Params::BP)
        {
            bp->setNumIters(bp->getNumIters() + 1);
            cout << "iter_count: " << bp->getNumIters() << endl;
        }
        else if (p.method == Params::CSBP)
        {
            csbp->setNumIters(csbp->getNumIters() + 1);
            cout << "iter_count: " << csbp->getNumIters() << endl;
        }
        break;
    case 'e': case 'E':
        if (p.method == Params::BP)
        {
            bp->setNumIters(max(bp->getNumIters() - 1, 1));
            cout << "iter_count: " << bp->getNumIters() << endl;
        }
        else if (p.method == Params::CSBP)
        {
            csbp->setNumIters(max(csbp->getNumIters() - 1, 1));
            cout << "iter_count: " << csbp->getNumIters() << endl;
        }
        break;
    case '4':
        if (p.method == Params::BP)
        {
            bp->setNumLevels(bp->getNumLevels() + 1);
            cout << "level_count: " << bp->getNumLevels() << endl;
        }
        else if (p.method == Params::CSBP)
        {
            csbp->setNumLevels(csbp->getNumLevels() + 1);
            cout << "level_count: " << csbp->getNumLevels() << endl;
        }
        break;
    case 'r': case 'R':
        if (p.method == Params::BP)
        {
            bp->setNumLevels(max(bp->getNumLevels() - 1, 1));
            cout << "level_count: " << bp->getNumLevels() << endl;
        }
        else if (p.method == Params::CSBP)
        {
            csbp->setNumLevels(max(csbp->getNumLevels() - 1, 1));
            cout << "level_count: " << csbp->getNumLevels() << endl;
        }
        break;
    }
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

- **Params**: A class/struct defined in this file
- **App**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iomanip`
- `iostream`
- `opencv2/imgproc.hpp`
- `string`
- `sstream`
- `stdexcept`
- `opencv2/core/utility.hpp`
- `opencv2/cudastereo.hpp`
- `opencv2/highgui.hpp`


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

