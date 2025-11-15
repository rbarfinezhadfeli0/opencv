# Documentation for `modules/gapi/test/own/last_written_value_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/own/last_written_value_tests.cpp`
- **File Name**: `last_written_value_tests.cpp`
- **File Size**: 4,409 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/own/last_written_value_tests.cpp](../../../../modules/gapi/test/own/last_written_value_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/own` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation

#include "../test_precomp.hpp"

#include <unordered_set>
#include <thread>

#include "executor/last_value.hpp"

namespace opencv_test {
using namespace cv::gapi;

TEST(LastValue, PushPop) {
    own::last_written_value<int> v;
    for (int i = 0; i < 100; i++) {
        v.push(i);

        int x = 1;
        v.pop(x);
        EXPECT_EQ(i, x);
    }
}

TEST(LastValue, TryPop) {
    own::last_written_value<int> v;
    int x = 0;
    EXPECT_FALSE(v.try_pop(x));

    v.push(1);
    EXPECT_TRUE(v.try_pop(x));
    EXPECT_EQ(1, x);
}

TEST(LastValue, Clear) {
    own::last_written_value<int> v;
    v.push(42);
    v.clear();

    int x = 0;
    EXPECT_FALSE(v.try_pop(x));
}

TEST(LastValue, Overwrite) {
    own::last_written_value<int> v;
    v.push(42);
    v.push(0);

    int x = -1;
    EXPECT_TRUE(v.try_pop(x));
    EXPECT_EQ(0, x);
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
namespace {
using StressParam = std::tuple<int   // Num writer threads
                              ,int   // Num elements per writer
                              ,int>; // Num reader threads
constexpr int STOP_SIGN = -1;
constexpr int BASE      = 1000;
}
struct LastValue_: public ::testing::TestWithParam<StressParam> {
    using V = own::last_written_value<int>;
    using S = std::unordered_set<int>;

    static void writer(int base, int writes, V& v) {
        for (int i = 0; i < writes; i++) {
            if (i % 2) {
                std::this_thread::sleep_for(std::chrono::milliseconds{1});
            }
            v.push(base + i);
        }
        v.push(STOP_SIGN);
    }

    static void reader(V& v, S& s) {
        int x = 0;
        while (true) {
            v.pop(x);
            if (x == STOP_SIGN) {
                // If this thread was lucky enough to read this STOP_SIGN,
                // push it back to v to make other possible readers able
                // to read it again (note due to the last_written_value
                // semantic, those STOP_SIGN could be simply lost i.e.
                // overwritten.
                v.push(STOP_SIGN);
                return;
            }
            s.insert(x);
        }
    }
};

TEST_P(LastValue_, Test)
{
    int num_writers = 0;
    int num_writes  = 0;
    int num_readers = 0;
    std::tie(num_writers, num_writes, num_readers) = GetParam();

    CV_Assert(num_writers <   20);
    CV_Assert(num_writes  < BASE);

    V v;

    // Start reader threads
    std::vector<S> storage(num_readers);
    std::vector<std::thread> readers;
    for (S& s : storage) {
        readers.emplace_back(reader, std::ref(v), std::ref(s));
    }

    // Start writer threads, also pre-generate reference numbers
    S reference;
    std::vector<std::thread> writers;
    for (int w = 0; w < num_writers; w++) {
        writers.emplace_back(writer, w*BASE, num_writes, std::ref(v));
        for (int r = 0; r < num_writes; r++) {
            reference.insert(w*BASE + r);
        }
    }

    // Wait for completions
    for (auto &t : readers) t.join();
    for (auto &t : writers) t.join();

    // Validate the result. Some values are read, and the values are
    // correct (i.e. such values have been written)
    std::size_t num_values_read = 0u;
    for (const auto &s : storage) {
        num_values_read += s.size();
        for (auto &x : s) {
            EXPECT_TRUE(reference.count(x) > 0);
        }
    }
    // NOTE: Some combinations may end-up in 0 values read
    // it is normal, the main thing is that the test shouldn't hang!
    EXPECT_LE(0u, num_values_read);
}

INSTANTIATE_TEST_CASE_P(LastValueStress, LastValue_,
                        Combine( Values(1, 2, 4, 8, 16)  // writers
                               , Values(32, 96, 256)     // writes
                               , Values(1, 2, 10)));     // readers
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

- **LastValue_**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `unordered_set`
- `../test_precomp.hpp`
- `executor/last_value.hpp`
- `thread`

**Python Imports:**
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

