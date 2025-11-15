# Documentation for `samples/directx/d3d10_interop.cpp`

## File Metadata

- **Full Path**: `samples/directx/d3d10_interop.cpp`
- **File Name**: `d3d10_interop.cpp`
- **File Size**: 10,193 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/directx/d3d10_interop.cpp](../../samples/directx/d3d10_interop.cpp)

## Purpose and Role

This file is located in the `samples/directx` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
// A sample program demonstrating interoperability of OpenCV cv::UMat with Direct X surface
// At first, the data obtained from video file or camera and placed onto Direct X surface,
// following mapping of this Direct X surface to OpenCV cv::UMat and call cv::Blur function.
// The result is mapped back to Direct X surface and rendered through Direct X API.
*/

#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <d3d10.h>

#include "opencv2/core.hpp"
#include "opencv2/core/directx.hpp"
#include "opencv2/core/ocl.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/videoio.hpp"

#include "d3dsample.hpp"


class D3D10WinApp : public D3DSample
{
public:
    D3D10WinApp(int width, int height, std::string& window_name, cv::VideoCapture& cap) :
        D3DSample(width, height, window_name, cap) {}

    ~D3D10WinApp() {}


    int create(void)
    {
        // base initialization
        D3DSample::create();

        // initialize DirectX
        HRESULT r;

        DXGI_SWAP_CHAIN_DESC scd;

        ZeroMemory(&scd, sizeof(DXGI_SWAP_CHAIN_DESC));

        scd.BufferCount       = 1;                               // one back buffer
        scd.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;      // use 32-bit color
        scd.BufferDesc.Width  = m_width;                         // set the back buffer width
        scd.BufferDesc.Height = m_height;                        // set the back buffer height
        scd.BufferUsage       = DXGI_USAGE_RENDER_TARGET_OUTPUT; // how swap chain is to be used
        scd.OutputWindow      = m_hWnd;                          // the window to be used
        scd.SampleDesc.Count  = 1;                               // how many multisamples
        scd.Windowed          = TRUE;                            // windowed/full-screen mode
        scd.SwapEffect        = DXGI_SWAP_EFFECT_DISCARD;
        scd.Flags             = DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH; // allow full-screen switching

        r = ::D3D10CreateDeviceAndSwapChain(
                NULL,
                D3D10_DRIVER_TYPE_HARDWARE,
                NULL,
                0,
                D3D10_SDK_VERSION,
                &scd,
                &m_pD3D10SwapChain,
                &m_pD3D10Dev);
        if (FAILED(r))
        {
            return EXIT_FAILURE;
        }

        r = m_pD3D10SwapChain->GetBuffer(0, __uuidof(ID3D10Texture2D), (LPVOID*)&m_pBackBuffer);
        if (FAILED(r))
        {
            return EXIT_FAILURE;
        }

        r = m_pD3D10Dev->CreateRenderTargetView(m_pBackBuffer, NULL, &m_pRenderTarget);
        if (FAILED(r))
        {
            return EXIT_FAILURE;
        }

        m_pD3D10Dev->OMSetRenderTargets(1, &m_pRenderTarget, NULL);

        D3D10_VIEWPORT viewport;
        ZeroMemory(&viewport, sizeof(D3D10_VIEWPORT));

        viewport.Width    = m_width;
        viewport.Height   = m_height;
        viewport.MinDepth = 0.0f;
        viewport.MaxDepth = 0.0f;

        m_pD3D10Dev->RSSetViewports(1, &viewport);

        D3D10_TEXTURE2D_DESC desc = { 0 };

        desc.Width            = m_width;
        desc.Height           = m_height;
        desc.MipLevels        = 1;
        desc.ArraySize        = 1;
        desc.Format           = DXGI_FORMAT_R8G8B8A8_UNORM;
        desc.SampleDesc.Count = 1;
        desc.BindFlags        = D3D10_BIND_SHADER_RESOURCE;
        desc.Usage            = D3D10_USAGE_DYNAMIC;
        desc.CPUAccessFlags   = D3D10_CPU_ACCESS_WRITE;

        r = m_pD3D10Dev->CreateTexture2D(&desc, NULL, &m_pSurface);
        if (FAILED(r))
        {
            std::cerr << "Can't create texture with input image" << std::endl;
            return EXIT_FAILURE;
        }

        // initialize OpenCL context of OpenCV lib from DirectX
        if (cv::ocl::haveOpenCL())
        {
            m_oclCtx = cv::directx::ocl::initializeContextFromD3D10Device(m_pD3D10Dev);
        }

        m_oclDevName = cv::ocl::useOpenCL() ?
            cv::ocl::Context::getDefault().device(0).name() :
            "No OpenCL device";

        return EXIT_SUCCESS;
    } // create()


    // get media data on DX surface for further processing
    int get_surface(ID3D10Texture2D** ppSurface)
    {
        HRESULT r;

        if (!m_cap.read(m_frame_bgr))
            return EXIT_FAILURE;

        cv::cvtColor(m_frame_bgr, m_frame_rgba, cv::COLOR_BGR2RGBA);

        UINT subResource = ::D3D10CalcSubresource(0, 0, 1);

        D3D10_MAPPED_TEXTURE2D mappedTex;
        r = m_pSurface->Map(subResource, D3D10_MAP_WRITE_DISCARD, 0, &mappedTex);
        if (FAILED(r))
        {
            return r;
        }

        cv::Mat m(m_height, m_width, CV_8UC4, mappedTex.pData, (int)mappedTex.RowPitch);
        // copy video frame data to surface
        m_frame_rgba.copyTo(m);

        m_pSurface->Unmap(subResource);

        *ppSurface = m_pSurface;

        return EXIT_SUCCESS;
    } // get_surface()


    // process and render media data
    int render()
    {
        try
        {
            if (m_shutdown)
                return EXIT_SUCCESS;

            // capture user input once
            MODE mode = (m_mode == MODE_GPU_NV12) ? MODE_GPU_RGBA : m_mode;

            HRESULT r;
            ID3D10Texture2D* pSurface;

            r = get_surface(&pSurface);
            if (FAILED(r))
            {
                return EXIT_FAILURE;
            }

            m_timer.reset();
            m_timer.start();

            switch (mode)
            {
                case MODE_CPU:
                {
                    // process video frame on CPU
                    UINT subResource = ::D3D10CalcSubresource(0, 0, 1);

                    D3D10_MAPPED_TEXTURE2D mappedTex;
                    r = pSurface->Map(subResource, D3D10_MAP_WRITE_DISCARD, 0, &mappedTex);
                    if (FAILED(r))
                    {
                        return r;
                    }

                    cv::Mat m(m_height, m_width, CV_8UC4, mappedTex.pData, (int)mappedTex.RowPitch);

                    if (m_demo_processing)
                    {
                        // blur D3D10 surface with OpenCV on CPU
                        cv::blur(m, m, cv::Size(15, 15));
                    }

                    m_timer.stop();

                    cv::String strMode = cv::format("mode: %s", m_modeStr[MODE_CPU].c_str());
                    cv::String strProcessing = m_demo_processing ? "blur frame" : "copy frame";
                    cv::String strTime = cv::format("time: %4.3f msec", m_timer.getTimeMilli());
                    cv::String strDevName = cv::format("OpenCL device: %s", m_oclDevName.c_str());

                    cv::putText(m, strMode, cv::Point(0, 20), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);
                    cv::putText(m, strProcessing, cv::Point(0, 40), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);
                    cv::putText(m, strTime, cv::Point(0, 60), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);
                    cv::putText(m, strDevName, cv::Point(0, 80), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);

                    pSurface->Unmap(subResource);

                    break;
                }

                case MODE_GPU_RGBA:
                {
                    // process video frame on GPU
                    cv::UMat u;

                    cv::directx::convertFromD3D10Texture2D(pSurface, u);

                    if (m_demo_processing)
                    {
                        // blur D3D10 surface with OpenCV on GPU with OpenCL
                        cv::blur(u, u, cv::Size(15, 15));
                    }

                    m_timer.stop();

                    cv::String strMode = cv::format("mode: %s", m_modeStr[MODE_GPU_RGBA].c_str());
                    cv::String strProcessing = m_demo_processing ? "blur frame" : "copy frame";
                    cv::String strTime = cv::format("time: %4.3f msec", m_timer.getTimeMilli());
                    cv::String strDevName = cv::format("OpenCL device: %s", m_oclDevName.c_str());

                    cv::putText(u, strMode, cv::Point(0, 20), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);
                    cv::putText(u, strProcessing, cv::Point(0, 40), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);
                    cv::putText(u, strTime, cv::Point(0, 60), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);
                    cv::putText(u, strDevName, cv::Point(0, 80), cv::FONT_HERSHEY_SIMPLEX, 0.8, cv::Scalar(0, 0, 200), 2);

                    cv::directx::convertToD3D10Texture2D(u, pSurface);

                    break;
                }

            } // switch

            // traditional DX render pipeline:
            //   BitBlt surface to backBuffer and flip backBuffer to frontBuffer
            m_pD3D10Dev->CopyResource(m_pBackBuffer, pSurface);

            // present the back buffer contents to the display
            // switch the back buffer and the front buffer
            r = m_pD3D10SwapChain->Present(0, 0);
            if (FAILED(r))
            {
                return EXIT_FAILURE;
            }
        } // try

        catch (const cv::Exception& e)
        {
            std::cerr << "Exception: " << e.what() << std::endl;
            return 10;
        }

        return EXIT_SUCCESS;
    } // render()


    int cleanup(void)
    {
        SAFE_RELEASE(m_pSurface);
        SAFE_RELEASE(m_pBackBuffer);
        SAFE_RELEASE(m_pD3D10SwapChain);
        SAFE_RELEASE(m_pRenderTarget);
        SAFE_RELEASE(m_pD3D10Dev);
        D3DSample::cleanup();
        return EXIT_SUCCESS;
    } // cleanup()

private:
    ID3D10Device*           m_pD3D10Dev;
    IDXGISwapChain*         m_pD3D10SwapChain;
    ID3D10Texture2D*        m_pBackBuffer;
    ID3D10Texture2D*        m_pSurface;
    ID3D10RenderTargetView* m_pRenderTarget;
    cv::ocl::Context        m_oclCtx;
    cv::String              m_oclPlatformName;
    cv::String              m_oclDevName;
};


// main func
int main(int argc, char** argv)
{
    std::string title = "D3D10 interop sample";
    return d3d_app<D3D10WinApp>(argc, argv, title);
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

- **D3D10WinApp**: A class/struct defined in this file

### Functions and Methods

- **int()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/directx.hpp`
- `opencv2/imgproc.hpp`
- `d3d10.h`
- `opencv2/videoio.hpp`
- `opencv2/core.hpp`
- `d3dsample.hpp`
- `opencv2/core/ocl.hpp`
- `windows.h`

**Python Imports:**
- `DirectX`
- `video`


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

