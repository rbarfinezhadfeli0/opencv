# Documentation for `modules/gapi/src/backends/fluid/gfluidbackend.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/fluid/gfluidbackend.hpp`
- **File Name**: `gfluidbackend.hpp`
- **File Size**: 6,180 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/backends/fluid/gfluidbackend.hpp](../../../../../modules/gapi/src/backends/fluid/gfluidbackend.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/fluid` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation

#ifndef OPENCV_GAPI_FLUID_BACKEND_HPP
#define OPENCV_GAPI_FLUID_BACKEND_HPP

// FIXME? Actually gfluidbackend.hpp is not included anywhere
// and can be placed in gfluidbackend.cpp

#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/gproto.hpp>
#include <opencv2/gapi/fluid/gfluidkernel.hpp>
#include <opencv2/gapi/fluid/gfluidbuffer.hpp>

// PRIVATE STUFF!
#include "backends/common/gbackend.hpp"
#include "compiler/gislandmodel.hpp"

namespace cv { namespace gimpl {

struct FluidUnit
{
    static const char *name() { return "FluidUnit"; }
    GFluidKernel k;
    gapi::fluid::BorderOpt border;
    int border_size;
    int window;
    std::vector<int> line_consumption;
    double ratio;
};

struct FluidUseOwnBorderBuffer
{
    static const char *name() { return "FluidUseOwnBorderBuffer"; }
    bool use;
};

struct FluidData
{
    static const char *name() { return "FluidData"; }

    // FIXME: This structure starts looking like "FluidBuffer" meta
    int  latency         = 0;
    int  skew            = 0;
    int  max_consumption = 1;
    int  border_size     = 0;
    int  lpi_write       = 1;
    bool internal        = false; // is node internal to any fluid island
    gapi::fluid::BorderOpt border;
};

struct agent_data_t {
     GFluidKernel::Kind  kind;
     ade::NodeHandle     nh;
     std::vector<int>    in_buffer_ids;
     std::vector<int>    out_buffer_ids;
 };

struct FluidAgent
{
public:
    virtual ~FluidAgent() = default;
    FluidAgent(const ade::Graph &g, ade::NodeHandle nh);

    GFluidKernel k;
    ade::NodeHandle op_handle; // FIXME: why it is here??//
    std::string op_name;

    // <  0 - not a buffer
    // >= 0 - a buffer with RcID
    std::vector<int> in_buffer_ids;
    std::vector<int> out_buffer_ids;

    cv::GArgs in_args;
    std::vector<cv::gapi::fluid::View>   in_views; // sparce list of IN views
    std::vector<cv::gapi::fluid::Buffer*> out_buffers;

    // FIXME Current assumption is that outputs have EQUAL SIZES
    int m_outputLines = 0;
    int m_producedLines = 0;

    // Execution methods
    void reset();
    bool canWork() const;
    bool canRead() const;
    bool canWrite() const;
    void doWork();
    bool done() const;

    void debug(std::ostream& os);

    // FIXME:
    // refactor (implement a more solid replacement or
    // drop this method completely)
    virtual void setRatio(double ratio) = 0;

private:
    // FIXME!!!
    // move to another class
    virtual int firstWindow(std::size_t inPort) const = 0;
    virtual std::pair<int,int> linesReadAndnextWindow(std::size_t inPort) const = 0;
};

//helper data structure for accumulating graph traversal/analysis data
struct FluidGraphInputData {

    std::vector<agent_data_t>               m_agents_data;
    std::vector<std::size_t>                m_scratch_users;
    std::unordered_map<int, std::size_t>    m_id_map;           // GMat id -> buffer idx map
    std::map<std::size_t, ade::NodeHandle>  m_all_gmat_ids;

    std::size_t                             m_mat_count;
};
//local helper function to traverse the graph once and pass the results to multiple instances of GFluidExecutable
FluidGraphInputData fluidExtractInputDataFromGraph(const ade::Graph &m_g, const std::vector<ade::NodeHandle> &nodes);

class GFluidExecutable final: public GIslandExecutable
{
    GFluidExecutable(const GFluidExecutable&) = delete;  // due std::unique_ptr in members list

    const ade::Graph &m_g;
    GModel::ConstGraph m_gm;

    std::vector<std::unique_ptr<FluidAgent>> m_agents;

    std::vector<FluidAgent*> m_script;

    cv::gimpl::Mag m_res;

    std::size_t m_num_int_buffers; // internal buffers counter (m_buffers - num_scratch)
    std::vector<std::size_t> m_scratch_users;

    std::unordered_map<int, std::size_t> m_id_map; // GMat id -> buffer idx map
    std::map<std::size_t, ade::NodeHandle> m_all_gmat_ids;

    std::vector<cv::gapi::fluid::Buffer> m_buffers;

    void bindInArg (const RcDesc &rc, const GRunArg &arg);
    void bindOutArg(const RcDesc &rc, const GRunArgP &arg);
    void packArg   (GArg &in_arg, const GArg &op_arg);

    void initBufferRois(std::vector<int>& readStarts, std::vector<cv::Rect>& rois, const std::vector<cv::Rect> &out_rois);
    void makeReshape(const std::vector<cv::Rect>& out_rois);
    std::size_t total_buffers_size() const;

public:
    virtual inline bool canReshape() const override { return true; }
    virtual void reshape(ade::Graph& g, const GCompileArgs& args) override;

    virtual void run(std::vector<InObj>  &&input_objs,
                     std::vector<OutObj> &&output_objs) override;

    using GIslandExecutable::run; // (IInput&, IOutput&) version

    void run(std::vector<InObj>  &input_objs,
             std::vector<OutObj> &output_objs);


     GFluidExecutable(const ade::Graph                          &g,
                      const FluidGraphInputData                 &graph_data,
                      const std::vector<cv::Rect>               &outputRois);
};


class GParallelFluidExecutable final: public GIslandExecutable {
    GParallelFluidExecutable(const GParallelFluidExecutable&) = delete;  // due std::unique_ptr in members list

    std::vector<std::unique_ptr<GFluidExecutable>> tiles;
    decltype(GFluidParallelFor::parallel_for) parallel_for;
public:
    GParallelFluidExecutable(const ade::Graph                       &g,
                             const FluidGraphInputData              &graph_data,
                             const std::vector<GFluidOutputRois>    &parallelOutputRois,
                             const decltype(parallel_for)           &pfor);


    virtual inline bool canReshape() const override { return false; }
    virtual void reshape(ade::Graph& g, const GCompileArgs& args) override;

    virtual void run(std::vector<InObj>  &&input_objs,
                     std::vector<OutObj> &&output_objs) override;
};
}} // cv::gimpl


#endif // OPENCV_GAPI_FLUID_BACKEND_HPP
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

- **agent_data_t**: A class/struct defined in this file
- **virtual**: A class/struct defined in this file
- **FluidAgent**: A class/struct defined in this file
- **FluidGraphInputData**: A class/struct defined in this file
- **FluidData**: A class/struct defined in this file
- **FluidUseOwnBorderBuffer**: A class/struct defined in this file
- **GFluidExecutable**: A class/struct defined in this file
- **GParallelFluidExecutable**: A class/struct defined in this file
- **FluidUnit**: A class/struct defined in this file

### Functions and Methods

- **to()**: A function/method defined in this file
- **OPENCV_GAPI_FLUID_BACKEND_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `backends/common/gbackend.hpp`
- `opencv2/gapi/fluid/gfluidbuffer.hpp`
- `opencv2/gapi/garg.hpp`
- `compiler/gislandmodel.hpp`
- `opencv2/gapi/gproto.hpp`
- `opencv2/gapi/fluid/gfluidkernel.hpp`


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

