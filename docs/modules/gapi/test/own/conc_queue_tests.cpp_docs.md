# Documentation for `modules/gapi/test/own/conc_queue_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/own/conc_queue_tests.cpp`
- **File Name**: `conc_queue_tests.cpp`
- **File Size**: 5,282 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/own/conc_queue_tests.cpp](../../../../modules/gapi/test/own/conc_queue_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/own` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation

#include "../test_precomp.hpp"

#include <unordered_set>
#include <thread>

#include "executor/conc_queue.hpp"

namespace opencv_test
{
using namespace cv::gapi;

TEST(ConcQueue, PushPop)
{
    own::concurrent_bounded_queue<int> q;
    for (int i = 0; i < 100; i++)
    {
        q.push(i);
    }

    for (int i = 0; i < 100; i++)
    {
        int x;
        q.pop(x);
        EXPECT_EQ(i, x);
    }
}

TEST(ConcQueue, TryPop)
{
    own::concurrent_bounded_queue<int> q;
    int x = 0;
    EXPECT_FALSE(q.try_pop(x));

    q.push(1);
    EXPECT_TRUE(q.try_pop(x));
    EXPECT_EQ(1, x);
}

TEST(ConcQueue, Clear)
{
    own::concurrent_bounded_queue<int> q;
    for (int i = 0; i < 10; i++)
    {
        q.push(i);
    }

    q.clear();
    int x = 0;
    EXPECT_FALSE(q.try_pop(x));
}

// In this test, every writer thread produces its own range of integer
// numbers, writing those to a shared queue.
//
// Every reader thread pops elements from the queue (until -1 is
// reached) and stores those in its own associated set.
//
// Finally, the master thread waits for completion of all other
// threads and verifies that all the necessary data is
// produced/obtained.
namespace
{
using StressParam = std::tuple<int           // Num writer threads
                              ,int           // Num elements per writer
                              ,int           // Num reader threads
                              ,std::size_t>; // Queue capacity
constexpr int STOP_SIGN = -1;
constexpr int BASE      = 1000;
}
struct ConcQueue_: public ::testing::TestWithParam<StressParam>
{
    using Q = own::concurrent_bounded_queue<int>;
    using S = std::unordered_set<int>;

    static void writer(int base, int writes, Q& q)
    {
        for (int i = 0; i < writes; i++)
        {
            q.push(base + i);
        }
        q.push(STOP_SIGN);
    }

    static void reader(Q& q, S& s)
    {
        int x = 0;
        while (true)
        {
            q.pop(x);
            if (x == STOP_SIGN) return;
            s.insert(x);
        }
    }
};

TEST_P(ConcQueue_, Test)
{
    int num_writers = 0;
    int num_writes  = 0;
    int num_readers = 0;
    std::size_t capacity = 0u;
    std::tie(num_writers, num_writes, num_readers, capacity) = GetParam();

    CV_Assert(num_writers <   20);
    CV_Assert(num_writes  < BASE);

    Q q;
    if (capacity)
    {
        // see below (2)
        CV_Assert(static_cast<int>(capacity) > (num_writers - num_readers));
        q.set_capacity(capacity);
    }

    // Start reader threads
    std::vector<S> storage(num_readers);
    std::vector<std::thread> readers;
    for (S& s : storage)
    {
        readers.emplace_back(reader, std::ref(q), std::ref(s));
    }

    // Start writer threads, also pre-generate reference numbers
    S reference;
    std::vector<std::thread> writers;
    for (int w = 0; w < num_writers; w++)
    {
        writers.emplace_back(writer, w*BASE, num_writes, std::ref(q));
        for (int r = 0; r < num_writes; r++)
        {
            reference.insert(w*BASE + r);
        }
    }

    // Every writer puts a STOP_SIGN at the end,
    // There are three cases:
    // 1) num_writers == num_readers
    //    every reader should get its own STOP_SIGN from any
    //    of the writers
    //
    // 2) num_writers > num_readers
    //    every reader will get a STOP_SIGN but there're more
    //    STOP_SIGNs may be pushed to the queue - and if this
    //    number exceeds capacity, writers block (to a deadlock).
    //    The latter situation must be avoided at parameters level.
    //    [a] Also not every data produced by writers will be consumed
    //    by a reader in this case. Master thread will read the rest
    //
    // 3) num_readers > num_writers
    //    in this case, some readers will stuck and will never get
    //    a STOP_SIGN. Master thread will push extra STOP_SIGNs to the
    //    queue.

    // Solution to (2a)
    S remnants;
    if (num_writers > num_readers)
    {
        int extra = num_writers - num_readers;
        while (extra)
        {
            int x = 0;
            q.pop(x);
            if (x == STOP_SIGN) extra--;
            else remnants.insert(x);
        }
    }

    // Solution to (3)
    if (num_readers > num_writers)
    {
        int extra = num_readers - num_writers;
        while (extra--) q.push(STOP_SIGN);
    }

    // Wait for completions
    for (auto &t : readers) t.join();
    for (auto &t : writers) t.join();

    // Accumulate and validate the result
    S result(remnants.begin(), remnants.end());
    for (const auto &s : storage) result.insert(s.begin(), s.end());

    EXPECT_EQ(reference, result);
}

INSTANTIATE_TEST_CASE_P(ConcQueueStress, ConcQueue_,
                        Combine(  Values(1, 2, 4, 8, 16)     // writers
                                , Values(1, 32, 96, 256)     // writes
                                , Values(1, 2, 10)           // readers
                                , Values(0u, 16u, 32u)));    // capacity
} // namespace opencv_test
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

- **ConcQueue_**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `unordered_set`
- `../test_precomp.hpp`
- `thread`
- `executor/conc_queue.hpp`

**Python Imports:**
- `any`
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

