# Documentation for `modules/gapi/samples/pipeline_modeling_tool/pipeline.hpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/pipeline_modeling_tool/pipeline.hpp`
- **File Name**: `pipeline.hpp`
- **File Size**: 7,065 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/samples/pipeline_modeling_tool/pipeline.hpp](../../../../modules/gapi/samples/pipeline_modeling_tool/pipeline.hpp)

## Purpose and Role

This file is located in the `modules/gapi/samples/pipeline_modeling_tool` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef OPENCV_GAPI_PIPELINE_MODELING_TOOL_PIPELINE_HPP
#define OPENCV_GAPI_PIPELINE_MODELING_TOOL_PIPELINE_HPP

#include <iomanip>

struct PerfReport {
    std::string name;
    double  avg_latency        = 0.0;
    double  min_latency        = 0.0;
    double  max_latency        = 0.0;
    double  first_latency      = 0.0;
    double  throughput         = 0.0;
    double  elapsed            = 0.0;
    double  warmup_time        = 0.0;
    int64_t num_late_frames    = 0;
    std::vector<double>  latencies;
    std::vector<int64_t> seq_ids;

    std::string toStr(bool expanded = false) const;
};

std::string PerfReport::toStr(bool expand) const {
    const auto to_double_str = [](double val) {
        std::stringstream ss;
        ss << std::fixed << std::setprecision(3) << val;
        return ss.str();
    };

    std::stringstream ss;
    ss << name << ": warm-up: " << to_double_str(warmup_time)
       << " ms, execution time: " << to_double_str(elapsed)
       << " ms, throughput: " << to_double_str(throughput)
       << " FPS, latency: first: " << to_double_str(first_latency)
       << " ms, min: " << to_double_str(min_latency)
       << " ms, avg: " << to_double_str(avg_latency)
       << " ms, max: " << to_double_str(max_latency)
       << " ms, frames: " << num_late_frames << "/" << seq_ids.back()+1 << " (dropped/all)";
    if (expand) {
        for (size_t i = 0; i < latencies.size(); ++i) {
            ss << "\nFrame:" << i << "\nLatency: "
               << to_double_str(latencies[i]) << " ms";
        }
    }

    return ss.str();
}

class StopCriterion {
public:
    using Ptr = std::unique_ptr<StopCriterion>;

    virtual void start() = 0;
    virtual void iter()  = 0;
    virtual bool done()  = 0;
    virtual ~StopCriterion() = default;
};

class Pipeline {
public:
    using Ptr = std::shared_ptr<Pipeline>;

    Pipeline(std::string&&                  name,
             cv::GComputation&&             comp,
             std::shared_ptr<DummySource>&& src,
             StopCriterion::Ptr             stop_criterion,
             cv::GCompileArgs&&             args,
             const size_t                   num_outputs);

    void compile();
    void run();

    const PerfReport& report() const;
    const std::string& name() const { return m_name;}

    virtual ~Pipeline() = default;

protected:
    virtual void  _compile() = 0;
    virtual void  run_iter() = 0;
    virtual void  init() {};
    virtual void  deinit() {};

    void prepareOutputs();

    std::string                  m_name;
    cv::GComputation             m_comp;
    std::shared_ptr<DummySource> m_src;
    StopCriterion::Ptr           m_stop_criterion;
    cv::GCompileArgs             m_args;
    size_t                       m_num_outputs;
    PerfReport                   m_perf;

    cv::GRunArgsP                m_pipeline_outputs;
    std::vector<cv::Mat>         m_out_mats;
    int64_t                      m_start_ts;
    int64_t                      m_seq_id;
};

Pipeline::Pipeline(std::string&&                  name,
                   cv::GComputation&&             comp,
                   std::shared_ptr<DummySource>&& src,
                   StopCriterion::Ptr             stop_criterion,
                   cv::GCompileArgs&&             args,
                   const size_t                   num_outputs)
    : m_name(std::move(name)),
      m_comp(std::move(comp)),
      m_src(std::move(src)),
      m_stop_criterion(std::move(stop_criterion)),
      m_args(std::move(args)),
      m_num_outputs(num_outputs) {
    m_perf.name = m_name;
}

void Pipeline::compile() {
    m_perf.warmup_time =
        utils::measure<utils::double_ms_t>([this]() {
        _compile();
    });
}

void Pipeline::prepareOutputs() {
    // NB: N-2 buffers + timestamp + seq_id.
    m_out_mats.resize(m_num_outputs - 2);
    for (auto& m : m_out_mats) {
        m_pipeline_outputs += cv::gout(m);
    }
    m_pipeline_outputs += cv::gout(m_start_ts);
    m_pipeline_outputs += cv::gout(m_seq_id);
}

void Pipeline::run() {
    using namespace std::chrono;

    // NB: Allocate outputs for execution
    prepareOutputs();

    // NB: Warm-up iteration invalidates source state
    // so need to copy it
    auto orig_src = m_src;
    auto copy_src = std::make_shared<DummySource>(*m_src);

    // NB: Use copy for warm-up iteration
    m_src = copy_src;

    // NB: Warm-up iteration
    init();
    run_iter();
    deinit();

    // NB: Calculate first latency
    m_perf.first_latency = utils::double_ms_t{
        microseconds{utils::timestamp<microseconds>() - m_start_ts}}.count();

    // NB: Now use original source
    m_src = orig_src;

    // NB: Start measuring execution
    init();
    auto start = high_resolution_clock::now();
    m_stop_criterion->start();

    while (true) {
        run_iter();
        const auto latency = utils::double_ms_t{
            microseconds{utils::timestamp<microseconds>() - m_start_ts}}.count();

        m_perf.latencies.push_back(latency);
        m_perf.seq_ids.push_back(m_seq_id);

        m_stop_criterion->iter();

        if (m_stop_criterion->done()) {
            m_perf.elapsed = duration_cast<utils::double_ms_t>(
                    high_resolution_clock::now() - start).count();
            deinit();
            break;
        }
    }

    m_perf.avg_latency = utils::avg(m_perf.latencies);
    m_perf.min_latency = utils::min(m_perf.latencies);
    m_perf.max_latency = utils::max(m_perf.latencies);

    // NB: Count the number of dropped frames
    int64_t prev_seq_id = m_perf.seq_ids[0];
    for (size_t i = 1; i < m_perf.seq_ids.size(); ++i) {
        m_perf.num_late_frames += m_perf.seq_ids[i] - prev_seq_id - 1;
        prev_seq_id = m_perf.seq_ids[i];
    }

    m_perf.throughput = (m_perf.latencies.size() / m_perf.elapsed) * 1000;
}

const PerfReport& Pipeline::report() const {
    return m_perf;
}

class StreamingPipeline : public Pipeline {
public:
    using Pipeline::Pipeline;

private:
    void _compile() override {
        m_compiled =
            m_comp.compileStreaming({m_src->descr_of()},
                                     cv::GCompileArgs(m_args));
    }

    virtual void init() override {
        m_compiled.setSource(m_src);
        m_compiled.start();
    }

    virtual void deinit() override {
        m_compiled.stop();
    }

    virtual void run_iter() override {
        m_compiled.pull(cv::GRunArgsP{m_pipeline_outputs});
    }

    cv::GStreamingCompiled m_compiled;
};

class RegularPipeline : public Pipeline {
public:
    using Pipeline::Pipeline;

private:
    void _compile() override {
        m_compiled =
            m_comp.compile({m_src->descr_of()},
                            cv::GCompileArgs(m_args));
    }

    virtual void run_iter() override {
        cv::gapi::wip::Data data;
        m_src->pull(data);
        m_compiled({data}, cv::GRunArgsP{m_pipeline_outputs});
    }

    cv::GCompiled m_compiled;
};

enum class PLMode {
    REGULAR,
    STREAMING
};

#endif // OPENCV_GAPI_PIPELINE_MODELING_TOOL_PIPELINE_HPP
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

- **StopCriterion**: A class/struct defined in this file
- **StreamingPipeline**: A class/struct defined in this file
- **PLMode**: A class/struct defined in this file
- **PerfReport**: A class/struct defined in this file
- **RegularPipeline**: A class/struct defined in this file
- **Pipeline**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_PIPELINE_MODELING_TOOL_PIPELINE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iomanip`


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

