# Documentation for `samples/cpp/qrcode.cpp`

## File Metadata

- **Full Path**: `samples/cpp/qrcode.cpp`
- **File Name**: `qrcode.cpp`
- **File Size**: 10,402 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/qrcode.cpp](../../samples/cpp/qrcode.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "opencv2/objdetect.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/imgcodecs.hpp"
#include <string>
#include <iostream>

using namespace std;
using namespace cv;

static int liveQRCodeDetect();
static int imageQRCodeDetect(const string& in_file);

static bool g_useArucoBased = false;
static bool g_modeMultiQR = false;
static bool g_detectOnly = false;

static string g_out_file_name, g_out_file_ext;
static int g_save_idx = 0;

static bool g_saveDetections = false;
static bool g_saveAll = false;

static string getQRModeString()
{
    std::ostringstream out;
    out << "QR"
        << (g_modeMultiQR ? " multi" : "")
        << (g_detectOnly ? " detector" : " decoder");
    return out.str();
}

int main(int argc, char *argv[])
{
    const string keys =
        "{h help ? |        | print help messages }"
        "{i in     |        | input image path (also switches to image detection mode) }"
        "{aruco_based | false | use Aruco-based QR code detector instead of contour-based }"
        "{detect   | false  | detect QR code only (skip decoding) }"
        "{m multi  |        | use detect for multiple qr-codes }"
        "{o out    | qr_code.png | path to result file }"
        "{save_detections | false  | save all QR detections (video mode only) }"
        "{save_all | false  | save all processed frames  (video mode only) }"
    ;
    CommandLineParser cmd_parser(argc, argv, keys);

    cmd_parser.about("This program detects the QR-codes from camera or images using the OpenCV library.");
    if (cmd_parser.has("help"))
    {
        cmd_parser.printMessage();
        return 0;
    }

    string in_file_name = cmd_parser.get<string>("in");    // path to input image

    if (cmd_parser.has("out"))
    {
        std::string fpath = cmd_parser.get<string>("out");   // path to output image
        std::string::size_type idx = fpath.rfind('.');
        if (idx != std::string::npos)
        {
            g_out_file_name = fpath.substr(0, idx);
            g_out_file_ext = fpath.substr(idx);
        }
        else
        {
            g_out_file_name = fpath;
            g_out_file_ext = ".png";
        }
    }
    if (!cmd_parser.check())
    {
        cmd_parser.printErrors();
        return -1;
    }

    g_modeMultiQR = cmd_parser.has("multi") && cmd_parser.get<bool>("multi");
    g_detectOnly = cmd_parser.has("detect") && cmd_parser.get<bool>("detect");
    g_useArucoBased = cmd_parser.has("aruco_based") && cmd_parser.get<bool>("aruco_based");

    g_saveDetections = cmd_parser.has("save_detections") && cmd_parser.get<bool>("save_detections");
    g_saveAll = cmd_parser.has("save_all") && cmd_parser.get<bool>("save_all");

    int return_code = 0;
    if (in_file_name.empty())
    {
        return_code = liveQRCodeDetect();
    }
    else
    {
        return_code = imageQRCodeDetect(samples::findFile(in_file_name));
    }
    return return_code;
}

static
void drawQRCodeContour(Mat &color_image, const vector<Point>& corners)
{
    if (!corners.empty())
    {
        double show_radius = (color_image.rows  > color_image.cols)
                   ? (2.813 * color_image.rows) / color_image.cols
                   : (2.813 * color_image.cols) / color_image.rows;
        double contour_radius = show_radius * 0.4;

        vector< vector<Point> > contours;
        contours.push_back(corners);
        drawContours(color_image, contours, 0, Scalar(211, 0, 148), cvRound(contour_radius));

        RNG rng(1000);
        for (size_t i = 0; i < 4; i++)
        {
            Scalar color = Scalar(rng.uniform(0,255), rng.uniform(0, 255), rng.uniform(0, 255));
            circle(color_image, corners[i], cvRound(show_radius), color, -1);
        }
    }
}

static
void drawFPS(Mat &color_image, double fps)
{
    ostringstream convert;
    convert << cv::format("%.2f", fps) << " FPS (" << getQRModeString() << ")";
    putText(color_image, convert.str(), Point(25, 25), FONT_HERSHEY_DUPLEX, 1, Scalar(0, 0, 255), 2);
}

static
void drawQRCodeResults(Mat& frame, const vector<Point>& corners, const vector<cv::String>& decode_info, double fps)
{
    if (!corners.empty())
    {
        for (size_t i = 0; i < corners.size(); i += 4)
        {
            size_t qr_idx = i / 4;
            vector<Point> qrcode_contour(corners.begin() + i, corners.begin() + i + 4);
            drawQRCodeContour(frame, qrcode_contour);

            cout << "QR[" << qr_idx << "] @ " << Mat(qrcode_contour).reshape(2, 1) << ": ";
            if (decode_info.size() > qr_idx)
            {
                if (!decode_info[qr_idx].empty())
                    cout << "'" << decode_info[qr_idx] << "'" << endl;
                else
                    cout << "can't decode QR code" << endl;
            }
            else
            {
                cout << "decode information is not available (disabled)" << endl;
            }
        }
    }
    else
    {
        cout << "QR code is not detected" << endl;
    }

    drawFPS(frame, fps);
}

static
void runQR(
    const GraphicalCodeDetector& qrcode, const Mat& input,
    vector<Point>& corners, vector<cv::String>& decode_info
    // +global: bool g_modeMultiQR, bool g_detectOnly
)
{
    if (!g_modeMultiQR)
    {
        if (!g_detectOnly)
        {
            String decode_info1 = qrcode.detectAndDecode(input, corners);
            decode_info.push_back(decode_info1);
        }
        else
        {
            bool detection_result = qrcode.detect(input, corners);
            CV_UNUSED(detection_result);
        }
    }
    else
    {
        if (!g_detectOnly)
        {
            bool result_detection = qrcode.detectAndDecodeMulti(input, decode_info, corners);
            CV_UNUSED(result_detection);
        }
        else
        {
            bool result_detection = qrcode.detectMulti(input, corners);
            CV_UNUSED(result_detection);
        }
    }
}

static
double processQRCodeDetection(const GraphicalCodeDetector& qrcode, const Mat& input, Mat& result, vector<Point>& corners)
{
    if (input.channels() == 1)
        cvtColor(input, result, COLOR_GRAY2BGR);
    else
        input.copyTo(result);

    cout << "Run " << getQRModeString()
        << " on image: " << input.size() << " (" << typeToString(input.type()) << ")"
        << endl;

    TickMeter timer;

    vector<cv::String> decode_info;
    timer.start();
    runQR(qrcode, input, corners, decode_info);
    timer.stop();

    double fps = 1 / timer.getTimeSec();
    drawQRCodeResults(result, corners, decode_info, fps);

    return fps;
}

int liveQRCodeDetect()
{
    VideoCapture cap(0);

    if (!cap.isOpened())
    {
        cout << "Cannot open a camera" << endl;
        return 2;
    }

    cout << "Press 'm' to switch between detectAndDecode and detectAndDecodeMulti" << endl;
    cout << "Press 'd' to switch between decoder and detector" << endl;
    cout << "Press ' ' (space) to save result into images" << endl;
    cout << "Press 'ESC' to exit" << endl;
    GraphicalCodeDetector qrcode = QRCodeDetector();
    if (g_useArucoBased)
        qrcode = QRCodeDetectorAruco();

    for (;;)
    {
        Mat frame;
        cap >> frame;
        if (frame.empty())
        {
            cout << "End of video stream" << endl;
            break;
        }

        bool forceSave = g_saveAll;

        Mat result;

        try
        {
            vector<Point> corners;
            double fps = processQRCodeDetection(qrcode, frame, result, corners);
            cout << "FPS: " << fps << endl;
            forceSave |= (g_saveDetections && !corners.empty());
            //forceSave |= fps < 1.0;
        }
        catch (const cv::Exception& e)
        {
            cerr << "ERROR exception: " << e.what() << endl;
            forceSave = true;
        }

        if (!result.empty())
            imshow("QR code", result);

        int code = waitKey(1);
        if (code < 0 && !forceSave)
            continue; // timeout
        char c = (char)code;
        if (c == ' ' || forceSave)
        {
            string fsuffix = cv::format("-%05d", g_save_idx++);

            string fname_input = g_out_file_name + fsuffix + "_input.png";
            cout << "Saving QR code detection input: '" << fname_input << "' ..." << endl;
            imwrite(fname_input, frame);

            string fname = g_out_file_name + fsuffix + g_out_file_ext;
            cout << "Saving QR code detection result: '" << fname << "' ..." << endl;
            imwrite(fname, result);

            cout << "Saved" << endl;
        }
        if (c == 'm')
        {
            g_modeMultiQR = !g_modeMultiQR;
            cout << "Switching QR code mode ==> " << (g_modeMultiQR ? "detectAndDecodeMulti" : "detectAndDecode") << endl;
        }
        if (c == 'd')
        {
            g_detectOnly = !g_detectOnly;
            cout << "Switching QR decoder mode ==> " << (g_detectOnly ? "detect" : "decode") << endl;
        }
        if (c == 27)
        {
            cout << "'ESC' is pressed. Exiting..." << endl;
            break;
        }
    }
    cout << "Exit." << endl;

    return 0;
}

int imageQRCodeDetect(const string& in_file)
{
    const int count_experiments = 10;

    Mat input = imread(in_file, IMREAD_COLOR);
    cout << "Run " << getQRModeString()
        << " on image: " << input.size() << " (" << typeToString(input.type()) << ")"
        << endl;

    GraphicalCodeDetector qrcode = QRCodeDetector();
    if (g_useArucoBased)
        qrcode = QRCodeDetectorAruco();

    vector<Point> corners;
    vector<cv::String> decode_info;

    TickMeter timer;
    for (size_t i = 0; i < count_experiments; i++)
    {
        corners.clear();
        decode_info.clear();

        timer.start();
        runQR(qrcode, input, corners, decode_info);
        timer.stop();
    }
    double fps = count_experiments / timer.getTimeSec();
    cout << "FPS: " << fps << endl;

    Mat result; input.copyTo(result);
    drawQRCodeResults(result, corners, decode_info, fps);

    imshow("QR", result); waitKey(1);

    if (!g_out_file_name.empty())
    {
        string out_file = g_out_file_name + g_out_file_ext;
        cout << "Saving result: " << out_file << endl;
        imwrite(out_file, result);
    }

    cout << "Press any key to exit ..." << endl;
    waitKey(0);
    cout << "Exit." << endl;

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
- `opencv2/imgcodecs.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `string`
- `opencv2/videoio.hpp`
- `opencv2/highgui.hpp`
- `opencv2/objdetect.hpp`

**Python Imports:**
- `camera`


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

