# Documentation for `modules/videoio/src/cap_obsensor/obsensor_stream_channel_msmf.hpp`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_obsensor/obsensor_stream_channel_msmf.hpp`
- **File Name**: `obsensor_stream_channel_msmf.hpp`
- **File Size**: 4,677 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/videoio/src/cap_obsensor/obsensor_stream_channel_msmf.hpp](../../../../modules/videoio/src/cap_obsensor/obsensor_stream_channel_msmf.hpp)

## Purpose and Role

This file is located in the `modules/videoio/src/cap_obsensor` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

/*
* Copyright(C) 2022 by ORBBEC Technology., Inc.
* Authors:
*   Huang Zhenchang <yufeng@orbbec.com>
*
* Licensed under the Apache License, Version 2.0 (the "License");
* you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
*     http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/

#ifndef OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_MSMF_HPP
#define OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_MSMF_HPP
#ifdef HAVE_OBSENSOR_MSMF

#include "obsensor_uvc_stream_channel.hpp"

#include <condition_variable>

#include <windows.h>
#include <guiddef.h>
#include <mfapi.h>
#include <mfidl.h>
#include <mfplay.h>
#include <mfobjects.h>
#include <mfreadwrite.h>
#include <tchar.h>
#include <strsafe.h>
#include <codecvt>
#include <ks.h>
#include <comdef.h>
#include <mutex>
#include <vidcap.h> //IKsTopologyInfo
#include <ksproxy.h> //IKsControl
#include <ksmedia.h>

namespace cv {
namespace obsensor {
template <class T>
class ComPtr
{
public:
    ComPtr() {}
    ComPtr(T* lp)
    {
        p = lp;
    }
    ComPtr(_In_ const ComPtr<T>& lp)
    {
        p = lp.p;
    }
    virtual ~ComPtr() {}

    void swap(_In_ ComPtr<T>& lp)
    {
        ComPtr<T> tmp(p);
        p = lp.p;
        lp.p = tmp.p;
        tmp = NULL;
    }
    T** operator&()
    {
        CV_Assert(p == NULL);
        return p.operator&();
    }
    T* operator->() const
    {
        CV_Assert(p != NULL);
        return p.operator->();
    }
    operator bool()
    {
        return p.operator!=(NULL);
    }

    T* Get() const
    {
        return p;
    }

    void Release()
    {
        if (p)
            p.Release();
    }

    // query for U interface
    template <typename U>
    HRESULT As(_Out_ ComPtr<U>& lp) const
    {
        lp.Release();
        return p->QueryInterface(__uuidof(U), reinterpret_cast<void**>((T**)&lp));
    }

private:
    _COM_SMARTPTR_TYPEDEF(T, __uuidof(T));
    TPtr p;
};

class MFContext
{
public:
    ~MFContext(void);
    static MFContext& getInstance();

    std::vector<UvcDeviceInfo> queryUvcDeviceInfoList();
    Ptr<IStreamChannel> createStreamChannel(const UvcDeviceInfo& devInfo);

private:
    MFContext(void);
};

struct FrameRate
{
    unsigned int denominator;
    unsigned int numerator;
};

class MSMFStreamChannel : public IUvcStreamChannel, public IMFSourceReaderCallback
{
public:
    MSMFStreamChannel(const UvcDeviceInfo& devInfo);
    virtual ~MSMFStreamChannel() noexcept;

    virtual void start(const StreamProfile& profile, FrameCallback frameCallback) override;
    virtual void stop() override;

private:
    virtual bool setXu(uint8_t ctrl, const uint8_t* data, uint32_t len) override;
    virtual bool getXu(uint8_t ctrl, uint8_t** data, uint32_t* len) override;

private:
    MFContext& mfContext_;

    ComPtr<IMFAttributes> deviceAttrs_ = nullptr;
    ComPtr<IMFMediaSource> deviceSource_ = nullptr;
    ComPtr<IMFAttributes> readerAttrs_ = nullptr;
    ComPtr<IMFSourceReader> streamReader_ = nullptr;
    ComPtr<IAMCameraControl> cameraControl_ = nullptr;
    ComPtr<IAMVideoProcAmp> videoProcAmp_ = nullptr;
    ComPtr<IKsTopologyInfo> xuKsTopologyInfo_ = nullptr;
    ComPtr<IUnknown> xuNodeInstance_ = nullptr;
    ComPtr<IKsControl> xuKsControl_ = nullptr;
    int xuNodeId_;

    FrameCallback frameCallback_;
    StreamProfile currentProfile_;
    int8_t currentStreamIndex_;

    StreamState streamState_;
    std::mutex streamStateMutex_;
    std::condition_variable streamStateCv_;

    std::vector<uint8_t> xuRecvBuf_;
    std::vector<uint8_t> xuSendBuf_;

public:
    STDMETHODIMP QueryInterface(REFIID iid, void** ppv) override;
    STDMETHODIMP_(ULONG)
        AddRef() override;
    STDMETHODIMP_(ULONG)
        Release() override;
    STDMETHODIMP OnReadSample(HRESULT /*hrStatus*/, DWORD dwStreamIndex, DWORD /*dwStreamFlags*/, LONGLONG /*llTimestamp*/, IMFSample* sample) override;
    STDMETHODIMP OnEvent(DWORD /*sidx*/, IMFMediaEvent* /*event*/) override;
    STDMETHODIMP OnFlush(DWORD) override;

private:
    long refCount_ = 1;
};
}} // namespace cv::obsensor::
#endif // HAVE_OBSENSOR_MSMF
#endif // OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_MSMF_HPP
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

- **FrameRate**: A class/struct defined in this file
- **MSMFStreamChannel**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **MFContext**: A class/struct defined in this file
- **ComPtr**: A class/struct defined in this file
- **template**: A class/struct defined in this file

### Functions and Methods

- **HAVE_OBSENSOR_MSMF()**: A function/method defined in this file
- **OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_MSMF_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `strsafe.h`
- `mfreadwrite.h`
- `mfidl.h`
- `guiddef.h`
- `ksproxy.h`
- `mfapi.h`
- `codecvt`
- `comdef.h`
- `vidcap.h`
- `mfobjects.h`
- `ksmedia.h`
- `obsensor_uvc_stream_channel.hpp`
- `ks.h`
- `tchar.h`
- `condition_variable`
- `mfplay.h`
- `windows.h`
- `mutex`


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

