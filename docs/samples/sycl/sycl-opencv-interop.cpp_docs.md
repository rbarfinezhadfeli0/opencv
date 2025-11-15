# Documentation for `samples/sycl/sycl-opencv-interop.cpp`

## File Metadata

- **Full Path**: `samples/sycl/sycl-opencv-interop.cpp`
- **File Name**: `sycl-opencv-interop.cpp`
- **File Size**: 9,342 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/sycl/sycl-opencv-interop.cpp](../../samples/sycl/sycl-opencv-interop.cpp)

## Purpose and Role

This file is located in the `samples/sycl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * The example of interoperability between SYCL/OpenCL and OpenCV.
 * - SYCL: https://www.khronos.org/sycl/
 * - SYCL runtime parameters: https://github.com/intel/llvm/blob/sycl/sycl/doc/EnvironmentVariables.md
 */
#include <CL/sycl.hpp>

#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/videoio.hpp>
#include <opencv2/imgproc.hpp>

#include <opencv2/core/ocl.hpp>


class sycl_inverse_kernel;  // can be omitted - modern SYCL versions doesn't require this

using namespace cv;


class App
{
public:
    App(const CommandLineParser& cmd);
    ~App();

    void initVideoSource();

    void initSYCL();

    void process_frame(cv::Mat& frame);

    /// to check result with CPU-only reference code
    Mat process_frame_reference(const cv::Mat& frame);

    int run();

    bool isRunning() { return m_running; }
    bool doProcess() { return m_process; }

    void setRunning(bool running)      { m_running = running; }
    void setDoProcess(bool process)    { m_process = process; }

protected:
    void handleKey(char key);

private:
    bool                        m_running;
    bool                        m_process;
    bool                        m_show_ui;

    int64                       m_t0;
    int64                       m_t1;
    float                       m_time;
    float                       m_frequency;

    std::string                 m_file_name;
    int                         m_camera_id;
    cv::VideoCapture            m_cap;
    cv::Mat                     m_frame;

    cl::sycl::queue sycl_queue;
};


App::App(const CommandLineParser& cmd)
{
    m_camera_id  = cmd.get<int>("camera");
    m_file_name  = cmd.get<std::string>("video");

    m_running    = false;
    m_process    = false;
} // ctor


App::~App()
{
    // nothing
}


void App::initSYCL()
{
    using namespace cl::sycl;

    // Configuration details: https://github.com/intel/llvm/blob/sycl/sycl/doc/EnvironmentVariables.md
    cl::sycl::default_selector selector;

    sycl_queue = cl::sycl::queue(selector, [](cl::sycl::exception_list l)
    {
        // exception_handler
        for (auto ep : l)
        {
            try
            {
                std::rethrow_exception(ep);
            }
            catch (const cl::sycl::exception& e)
            {
                std::cerr << "SYCL exception: " << e.what() << std::endl;
            }
        }
    });

    auto device = sycl_queue.get_device();
    auto platform = device.get_platform();
    std::cout << "SYCL device: " << device.get_info<info::device::name>()
        << " @ " << device.get_info<info::device::driver_version>()
        << " (platform: " << platform.get_info<info::platform::name>() << ")" << std::endl;

    if (device.is_host())
    {
        std::cerr << "SYCL can't select OpenCL device. Host is used for computations, interoperability is not available" << std::endl;
    }
    else
    {
        // bind OpenCL context/device/queue from SYCL to OpenCV
        try
        {
            auto ctx = cv::ocl::OpenCLExecutionContext::create(
                    platform.get_info<info::platform::name>(),
                    platform.get(),
                    sycl_queue.get_context().get(),
                    device.get()
                );
            ctx.bind();
        }
        catch (const cv::Exception& e)
        {
            std::cerr << "OpenCV: Can't bind SYCL OpenCL context/device/queue: " << e.what() << std::endl;
        }
        std::cout << "OpenCV uses OpenCL: " << (cv::ocl::useOpenCL() ? "True" : "False") << std::endl;
    }
} // initSYCL()


void App::initVideoSource()
{
    if (!m_file_name.empty() && m_camera_id == -1)
    {
        m_cap.open(samples::findFileOrKeep(m_file_name));
        if (!m_cap.isOpened())
            throw std::runtime_error(std::string("can't open video stream: ") + m_file_name);
    }
    else if (m_camera_id != -1)
    {
        m_cap.open(m_camera_id);
        if (!m_cap.isOpened())
            throw std::runtime_error(std::string("can't open camera: ") + std::to_string(m_camera_id));
    }
    else
        throw std::runtime_error(std::string("specify video source"));
} // initVideoSource()


void App::process_frame(cv::Mat& frame)
{
    using namespace cl::sycl;

    // cv::Mat => cl::sycl::buffer
    {
        CV_Assert(frame.isContinuous());
        CV_CheckTypeEQ(frame.type(), CV_8UC1, "");

        buffer<uint8_t, 2> frame_buffer(frame.data, range<2>(frame.rows, frame.cols));

        // done automatically: frame_buffer.set_write_back(true);

        sycl_queue.submit([&](handler& cgh) {
          auto pixels = frame_buffer.get_access<access::mode::read_write>(cgh);

          cgh.parallel_for<class sycl_inverse_kernel>(range<2>(frame.rows, frame.cols), [=](item<2> item) {
              uint8_t v = pixels[item];
              pixels[item] = ~v;
          });
        });

        sycl_queue.wait_and_throw();
    }

    // No way to extract cl_mem from cl::sycl::buffer (ref: 3.6.11 "Interfacing with OpenCL" of SYCL 1.2.1)
    // We just reusing OpenCL context/device/queue from SYCL here (see initSYCL() bind part) and call UMat processing
    {
        UMat blurResult;
        {
            UMat umat_buffer = frame.getUMat(ACCESS_RW);
            cv::blur(umat_buffer, blurResult, Size(3, 3));  // UMat doesn't support inplace
        }
        Mat result;
        blurResult.copyTo(result);
        swap(result, frame);
    }
}

Mat App::process_frame_reference(const cv::Mat& frame)
{
    Mat result;
    cv::bitwise_not(frame, result);
    Mat blurResult;
    cv::blur(result, blurResult, Size(3, 3));  // avoid inplace
    blurResult.copyTo(result);
    return result;
}

int App::run()
{
    std::cout << "Initializing..." << std::endl;

    initSYCL();
    initVideoSource();

    std::cout << "Press ESC to exit" << std::endl;
    std::cout << "      'p' to toggle ON/OFF processing" << std::endl;

    m_running = true;
    m_process = true;
    m_show_ui = true;

    int processedFrames = 0;

    cv::TickMeter timer;

    // Iterate over all frames
    while (isRunning() && m_cap.read(m_frame))
    {
        Mat m_frameGray;
        cvtColor(m_frame, m_frameGray, COLOR_BGR2GRAY);

        bool checkWithReference = (processedFrames == 0);
        Mat reference_result;
        if (checkWithReference)
        {
            reference_result = process_frame_reference(m_frameGray);
        }

        timer.reset();
        timer.start();

        if (m_process)
        {
            process_frame(m_frameGray);
        }

        timer.stop();

        if (checkWithReference)
        {
            double diffInf = cv::norm(reference_result, m_frameGray, NORM_INF);
            if (diffInf > 0)
            {
                std::cerr << "Result is not accurate. diffInf=" << diffInf << std::endl;
                imwrite("reference.png", reference_result);
                imwrite("actual.png", m_frameGray);
            }
        }

        Mat img_to_show = m_frameGray;

        std::ostringstream msg;
        msg << "Frame " << processedFrames << " (" << m_frame.size
            << ")   Time: " << cv::format("%.2f", timer.getTimeMilli()) << " msec"
            << " (process: " << (m_process ? "True" : "False") << ")";
        std::cout << msg.str() << std::endl;
        putText(img_to_show, msg.str(), Point(5, 150), FONT_HERSHEY_SIMPLEX, 1., Scalar(255, 100, 0), 2);

        if (m_show_ui)
        {
            try
            {
                imshow("sycl_interop", img_to_show);
                int key = waitKey(1);
                switch (key)
                {
                case 27:  // ESC
                    m_running = false;
                    break;

                case 'p':  // fallthru
                case 'P':
                    m_process = !m_process;
                    break;

                default:
                    break;
                }
            }
            catch (const std::exception& e)
            {
                std::cerr << "ERROR(OpenCV UI): " << e.what() << std::endl;
                if (processedFrames > 0)
                    throw;
                m_show_ui = false;  // UI is not available
            }
        }

        processedFrames++;

        if (!m_show_ui)
        {
            if (processedFrames > 100)
                m_running = false;
        }
    }

    return 0;
}


int main(int argc, char** argv)
{
    const char* keys =
        "{ help h ?    |          | print help message }"
        "{ camera c    | -1       | use camera as input }"
        "{ video  v    |          | use video as input }";

    CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help"))
    {
        cmd.printMessage();
        return EXIT_SUCCESS;
    }

    try
    {
        App app(cmd);
        if (!cmd.check())
        {
            cmd.printErrors();
            return 1;
        }
        app.run();
    }
    catch (const cv::Exception& e)
    {
        std::cout << "FATAL: OpenCV error: " << e.what() << std::endl;
        return 1;
    }
    catch (const std::exception& e)
    {
        std::cout << "FATAL: C++ error: " << e.what() << std::endl;
        return 1;
    }

    catch (...)
    {
        std::cout << "FATAL: unknown C++ exception" << std::endl;
        return 1;
    }

    return EXIT_SUCCESS;
} // main()
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

- **sycl_inverse_kernel**: A class/struct defined in this file
- **App**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `CL/sycl.hpp`
- `opencv2/imgproc.hpp`
- `opencv2/videoio.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`
- `opencv2/core/ocl.hpp`

**Python Imports:**
- `SYCL`
- `cl`


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

