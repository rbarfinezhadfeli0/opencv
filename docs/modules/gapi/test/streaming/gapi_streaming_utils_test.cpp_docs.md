# Documentation for `modules/gapi/test/streaming/gapi_streaming_utils_test.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/streaming/gapi_streaming_utils_test.cpp`
- **File Name**: `gapi_streaming_utils_test.cpp`
- **File Size**: 9,580 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/streaming/gapi_streaming_utils_test.cpp](../../../../modules/gapi/test/streaming/gapi_streaming_utils_test.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation


#include "../test_precomp.hpp"

#include "../common/gapi_streaming_tests_common.hpp"

#include <chrono>
#include <future>

#define private public
#include "streaming/onevpl/accelerators/utils/shared_lock.hpp"
#undef private

#include "streaming/onevpl/accelerators/utils/elastic_barrier.hpp"

namespace opencv_test
{
namespace
{
using cv::gapi::wip::onevpl::SharedLock;

struct TestBarrier : public cv::gapi::wip::onevpl::elastic_barrier<TestBarrier> {
    void on_first_in_impl(size_t visitor_id) {

        static std::atomic<int> thread_counter{};
        thread_counter++;
        EXPECT_EQ(thread_counter.load(), 1);

        visitors_in.insert(visitor_id);
        last_visitor_id = visitor_id;

        thread_counter--;
        EXPECT_EQ(thread_counter.load(), 0);
    }

    void on_last_out_impl(size_t visitor_id) {

        static std::atomic<int> thread_counter{};
        thread_counter++;
        EXPECT_EQ(thread_counter.load(), 1);

        visitors_out.insert(visitor_id);
        last_visitor_id = visitor_id;

        thread_counter--;
        EXPECT_EQ(thread_counter.load(), 0);
    }

    size_t last_visitor_id = 0;
    std::set<size_t> visitors_in;
    std::set<size_t> visitors_out;
};

TEST(OneVPL_SharedLock, Create) {
    SharedLock lock;
    EXPECT_EQ(lock.shared_counter.load(), size_t{0});
}

TEST(OneVPL_SharedLock, Read_SingleThread)
{
    SharedLock lock;

    const size_t single_thread_read_count = 100;
    for(size_t i = 0; i < single_thread_read_count; i++) {
        lock.shared_lock();
        EXPECT_FALSE(lock.owns());
    }
    EXPECT_EQ(lock.shared_counter.load(), single_thread_read_count);

    for(size_t i = 0; i < single_thread_read_count; i++) {
        lock.unlock_shared();
        EXPECT_FALSE(lock.owns());
    }

    EXPECT_EQ(lock.shared_counter.load(), size_t{0});
}

TEST(OneVPL_SharedLock, TryLock_SingleThread)
{
    SharedLock lock;

    EXPECT_TRUE(lock.try_lock());
    EXPECT_TRUE(lock.owns());

    lock.unlock();
    EXPECT_FALSE(lock.owns());
    EXPECT_EQ(lock.shared_counter.load(), size_t{0});
}

TEST(OneVPL_SharedLock, Write_SingleThread)
{
    SharedLock lock;

    lock.lock();
    EXPECT_TRUE(lock.owns());

    lock.unlock();
    EXPECT_FALSE(lock.owns());
    EXPECT_EQ(lock.shared_counter.load(), size_t{0});
}

TEST(OneVPL_SharedLock, TryLockTryLock_SingleThread)
{
    SharedLock lock;

    lock.try_lock();
    EXPECT_FALSE(lock.try_lock());
    lock.unlock();

    EXPECT_FALSE(lock.owns());
}

TEST(OneVPL_SharedLock, ReadTryLock_SingleThread)
{
    SharedLock lock;

    lock.shared_lock();
    EXPECT_FALSE(lock.owns());
    EXPECT_FALSE(lock.try_lock());
    lock.unlock_shared();

    EXPECT_TRUE(lock.try_lock());
    EXPECT_TRUE(lock.owns());
    lock.unlock();
}

TEST(OneVPL_SharedLock, WriteTryLock_SingleThread)
{
    SharedLock lock;

    lock.lock();
    EXPECT_TRUE(lock.owns());
    EXPECT_FALSE(lock.try_lock());
    lock.unlock();

    EXPECT_TRUE(lock.try_lock());
    EXPECT_TRUE(lock.owns());
    lock.unlock();
}


TEST(OneVPL_SharedLock, Write_MultiThread)
{
    SharedLock lock;

    std::promise<void> barrier;
    std::shared_future<void> sync = barrier.get_future();

    static const size_t inc_count = 10000000;
    size_t shared_value = 0;
    auto work = [&lock, &shared_value](size_t count) {
        for (size_t i = 0; i < count; i ++) {
            lock.lock();
            shared_value ++;
            lock.unlock();
        }
    };

    std::thread worker_thread([&barrier, sync, work] () {

        std::thread sub_worker([&barrier, work] () {
            barrier.set_value();
            work(inc_count);
        });

        sync.wait();
        work(inc_count);
        sub_worker.join();
    });
    sync.wait();

    work(inc_count);
    worker_thread.join();

    EXPECT_EQ(shared_value, inc_count * 3);
}

TEST(OneVPL_SharedLock, ReadWrite_MultiThread)
{
    SharedLock lock;

    std::promise<void> barrier;
    std::future<void> sync = barrier.get_future();

    static const size_t inc_count = 10000000;
    size_t shared_value = 0;
    auto write_work = [&lock, &shared_value](size_t count) {
        for (size_t i = 0; i < count; i ++) {
            lock.lock();
            shared_value ++;
            lock.unlock();
        }
    };

    auto read_work = [&lock, &shared_value](size_t count) {

        auto old_shared_value = shared_value;
        for (size_t i = 0; i < count; i ++) {
            lock.shared_lock();
            EXPECT_TRUE(shared_value >= old_shared_value);
            old_shared_value = shared_value;
            lock.unlock_shared();
        }
    };

    std::thread writer_thread([&barrier, write_work] () {
        barrier.set_value();
        write_work(inc_count);
    });
    sync.wait();

    read_work(inc_count);
    writer_thread.join();

    EXPECT_EQ(shared_value, inc_count);
}


TEST(OneVPL_ElasticBarrier, single_thread_visit)
{
    TestBarrier barrier;

    const size_t max_visit_count = 10000;
    size_t visit_id = 0;
    for (visit_id = 0; visit_id < max_visit_count; visit_id++) {
        barrier.visit_in(visit_id);
        EXPECT_EQ(barrier.visitors_in.size(), size_t{1});
    }
    EXPECT_EQ(barrier.last_visitor_id, size_t{0});
    EXPECT_EQ(barrier.visitors_out.size(), size_t{0});

    for (visit_id = 0; visit_id < max_visit_count; visit_id++) {
        barrier.visit_out(visit_id);
        EXPECT_EQ(barrier.visitors_in.size(), size_t{1});
    }
    EXPECT_EQ(barrier.last_visitor_id, visit_id - 1);
    EXPECT_EQ(barrier.visitors_out.size(), size_t{1});
}


TEST(OneVPL_ElasticBarrier, multi_thread_visit)
{
    applyTestTag(CV_TEST_TAG_VERYLONG);
    TestBarrier tested_barrier;

    static const size_t max_visit_count = 10000000;
    std::atomic<size_t> visit_in_wait_counter{};
    std::promise<void> start_sync_barrier;
    std::shared_future<void> start_sync = start_sync_barrier.get_future();
    std::promise<void> phase_sync_barrier;
    std::shared_future<void> phase_sync = phase_sync_barrier.get_future();

    auto visit_worker_job = [&tested_barrier,
                             &visit_in_wait_counter,
                             start_sync,
                             phase_sync] (size_t worker_id) {

        start_sync.wait();

        // first phase
        const size_t begin_range = worker_id * max_visit_count;
        const size_t end_range = (worker_id + 1) * max_visit_count;
        for (size_t visit_id = begin_range; visit_id < end_range; visit_id++) {
            tested_barrier.visit_in(visit_id);
        }

        // notify all worker first phase ready
        visit_in_wait_counter.fetch_add(1);

        // wait main second phase
        phase_sync.wait();

        // second phase
        for (size_t visit_id = begin_range; visit_id < end_range; visit_id++) {
            tested_barrier.visit_out(visit_id);
        }
    };

    auto visit_main_job = [&tested_barrier,
                           &visit_in_wait_counter,
                           &phase_sync_barrier] (size_t total_workers_count,
                                                 size_t worker_id) {

        const size_t begin_range = worker_id * max_visit_count;
        const size_t end_range = (worker_id + 1) * max_visit_count;
        for (size_t visit_id = begin_range; visit_id < end_range; visit_id++) {
            tested_barrier.visit_in(visit_id);
        }

        // wait all workers first phase done
        visit_in_wait_counter.fetch_add(1);
        while (visit_in_wait_counter.load() != total_workers_count) {
            std::this_thread::yield();
        };

        // TEST invariant: last_visitor_id MUST be one from any FIRST worker visitor_id
        bool one_of_available_ids_matched = false;
        for (size_t id = 0; id < total_workers_count; id ++) {
            size_t expected_last_visitor_for_id = id * max_visit_count;
            one_of_available_ids_matched |=
                    (tested_barrier.last_visitor_id == expected_last_visitor_for_id) ;
        }
        EXPECT_TRUE(one_of_available_ids_matched);

        // unblock all workers to work out second phase
        phase_sync_barrier.set_value();

        // continue second phase
        for (size_t visit_id = begin_range; visit_id < end_range; visit_id++) {
            tested_barrier.visit_out(visit_id);
        }
    };

    size_t max_worker_count = std::thread::hardware_concurrency();
    if (max_worker_count < 2) {
        max_worker_count = 2; // logical 2 threads required at least
    }
    std::vector<std::thread> workers;
    workers.reserve(max_worker_count);
    for (size_t worker_id = 1; worker_id < max_worker_count; worker_id++) {
        workers.emplace_back(visit_worker_job, worker_id);
    }

    // let's go for first phase
    start_sync_barrier.set_value();

    // utilize main thread as well
    visit_main_job(max_worker_count, 0);

    // join all threads second phase
    for (auto& w : workers) {
        w.join();
    }

    // TEST invariant: last_visitor_id MUST be one from any LATTER worker visitor_id
    bool one_of_available_ids_matched = false;
    for (size_t id = 0; id < max_worker_count; id ++) {
        one_of_available_ids_matched |=
                (tested_barrier.last_visitor_id == ((id + 1) * max_visit_count - 1)) ;
    }
    EXPECT_TRUE(one_of_available_ids_matched);
}
}
} // opencv_test
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

- **TestBarrier**: A class/struct defined in this file

### Functions and Methods

- **private()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../common/gapi_streaming_tests_common.hpp`
- `../test_precomp.hpp`
- `future`
- `chrono`
- `streaming/onevpl/accelerators/utils/elastic_barrier.hpp`
- `streaming/onevpl/accelerators/utils/shared_lock.hpp`

**Python Imports:**
- `any`


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

