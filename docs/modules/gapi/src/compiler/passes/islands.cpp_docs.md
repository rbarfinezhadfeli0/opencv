# Documentation for `modules/gapi/src/compiler/passes/islands.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/passes/islands.cpp`
- **File Name**: `islands.cpp`
- **File Size**: 9,083 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/compiler/passes/islands.cpp](../../../../../modules/gapi/src/compiler/passes/islands.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler/passes` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "precomp.hpp"

#include <sstream>
#include <stack>
#include <ade/util/chain_range.hpp>
#include <ade/graph.hpp>

#include "compiler/gmodel.hpp"
#include "compiler/passes/passes.hpp"

namespace
{
    bool is_within_same_island(const cv::gimpl::GModel::Graph &gr,
                               const ade::NodeHandle          &dataNode,
                               const std::string              &island)
    {
        // A data node is within the same island as it's reader node
        // if and only if data object's producer island (if there's a producer)
        // is the same as the specified one.
        //
        // An object may have only a single producer, but multiple consumers,
        // and these consumers may be assigned to different Islands.
        // Since "initIslands" traversal direction is op-to-args, i.e. reverse,
        // a single Data object may be visited twice during Islands initialization.
        //
        // In this case, Data object is part of Island A if and only if:
        // - Data object's producer is part of Island A,
        // - AND any of Data object's consumers is part of Island A.
        //
        //   Op["island0"] --> Data[ ? ] --> Op["island0"]
        //                       :
        //                       '---------> Op["island1"]
        //
        // In the above example, Data object is assigned to "island0" as
        // it is surrounded by operations assigned to "island0"

        using namespace cv::gimpl;

        if (   gr.metadata(dataNode).contains<Island>()
            && gr.metadata(dataNode).get<Island>().island != island)
            return false;

        if (dataNode->inNodes().empty())
            return false;

        GAPI_Assert(dataNode->inNodes().size() == 1u);
        const auto prod_h = dataNode->inNodes().front();

        // FIXME: ADE should have something like get_or<> or get<>(default)
        GAPI_Assert(gr.metadata(prod_h).get<NodeType>().t == NodeType::OP);
        return     (   gr.metadata(prod_h).contains<Island>()
                    && gr.metadata(prod_h).get<Island>().island == island)
                    && (ade::util::any_of(dataNode->outNodes(), [&](ade::NodeHandle cons_h)
                    {
                        return (   gr.metadata(cons_h).contains<Island>()
                                && gr.metadata(cons_h).get<Island>().island == island);
                    }));
    }
} // anonymous namespace

// Initially only Operations have Island tag. This pass adds Island tag
// to all data objects within an Island.
// A data object is considered within an Island if and only if
// its reader and writer are assigned to this Island (see above).
void cv::gimpl::passes::initIslands(ade::passes::PassContext &ctx)
{
    GModel::Graph gr(ctx.graph);
    for (const auto &nh : gr.nodes())
    {
        if (gr.metadata(nh).get<NodeType>().t == NodeType::OP)
        {
            if (gr.metadata(nh).contains<Island>())
            {
                const auto island = gr.metadata(nh).get<Island>().island;

                // It is enough to check only input nodes
                for (const auto &in_data_node : nh->inNodes())
                {
                    if (is_within_same_island(gr, in_data_node, island))
                    {
                        gr.metadata(in_data_node).set(Island{island});
                    }
                } // for(in_data_node)
            } // if (contains<Island>)
        } // if (OP)
    } // for (nodes)
}

// There should be no multiple (disconnected) islands with the same name.
// This may occur if user assigns the same islands name to multiple ranges
// in the graph.
// FIXME: How it could be avoided on an earlier stage?
void cv::gimpl::passes::checkIslands(ade::passes::PassContext &ctx)
{
    GModel::ConstGraph gr(ctx.graph);

    // The algorithm is the following:
    //
    // 1. Put all Tagged nodes (both Operations and Data) into a set
    // 2. Initialize Visited set as (empty)
    // 3. Initialize Traversal stack as (empty)
    // 4. Initialize Islands map (String -> Integer) as (empty)
    // 5. For every Tagged node from a set
    //    a. Skip if it is Visited
    //    b. For every input/output node:
    //       * if it is tagged with the same island:
    //         - add it to Traversal stack
    //         - remove from Tagged nodes if it is t
    //    c. While (stack is not empty):
    //       - Take a node from Stack
    //       - Repeat (b)
    //    d. Increment Islands map [this island] by 1
    //
    //
    // If whatever Island has counter is more than 1, it is a disjoint
    // one (i.e. there's two islands with the same name).

    using node_set = std::unordered_set
         < ade::NodeHandle
         , ade::HandleHasher<ade::Node>
         >;
    node_set tagged_nodes;
    node_set visited_tagged_nodes;
    std::unordered_map<std::string, int> island_counters;

    for (const auto &nh : gr.nodes())
    {
        if (gr.metadata(nh).contains<Island>())
        {
            tagged_nodes.insert(nh);
            island_counters[gr.metadata(nh).get<Island>().island] = 0;
        }
    }

    // Make a copy to allow list modifications during traversal
    for (const auto &tagged_nh : tagged_nodes)
    {
        if (visited_tagged_nodes.end() != ade::util::find(visited_tagged_nodes, tagged_nh))
            continue;

        // Run the recursive traversal process as described in 5/a-d.
        // This process is like a flood-fill traversal for island.
        // If there's to distinct successful flood-fills happened for the same island
        // name, there are two islands with this name.
        std::stack<ade::NodeHandle> stack;
        stack.push(tagged_nh);

        while (!stack.empty())
        {
            const auto this_nh = stack.top();
            stack.pop();

            // Since _this_ node is visited, it is a part of processed island
            // so mark it as visited to skip in other recursive processes
            visited_tagged_nodes.insert(this_nh);

            GAPI_DbgAssert(gr.metadata(this_nh).contains<Island>());
            GAPI_DbgAssert(   gr.metadata(this_nh  ).get<Island>().island
                         == gr.metadata(tagged_nh).get<Island>().island);
            const auto &this_island = gr.metadata(this_nh).get<Island>().island;

            for (const auto neighbor_nh : ade::util::chain(this_nh->inNodes(), this_nh->outNodes()))
            {
                if (   gr.metadata(neighbor_nh).contains<Island>()
                    && gr.metadata(neighbor_nh).get<Island>().island == this_island
                    && !visited_tagged_nodes.count(neighbor_nh))
                {
                    stack.push(neighbor_nh);
                }
            } // for (neighbor)
        } // while (stack)

        // Flood-fill is over, now increment island counter for this island
        island_counters[gr.metadata(tagged_nh).get<Island>().island]++;
    } // for(tagged)

    bool check_failed = false;
    std::stringstream ss;
    for (const auto &ic : island_counters)
    {
        GAPI_Assert(ic.second > 0);
        if (ic.second > 1)
        {
            check_failed = true;
            ss << "\"" << ic.first << "\"(" << ic.second << ") ";
        }
    }
    if (check_failed)
    {
        util::throw_error
            (std::logic_error("There are multiple distinct islands "
                              "with the same name: [" + ss.str() + "], "
                              "please check your cv::gapi::island() parameters!"));
    }
}

void cv::gimpl::passes::checkIslandsContent(ade::passes::PassContext &ctx)
{
    GModel::ConstGraph gr(ctx.graph);
    std::unordered_map<std::string, cv::gapi::GBackend> backends_of_islands;
    for (const auto& nh : gr.nodes())
    {
        if (NodeType::OP == gr.metadata(nh).get<NodeType>().t &&
            gr.metadata(nh).contains<Island>())
        {
            const auto island      = gr.metadata(nh).get<Island>().island;
            auto island_backend_it = backends_of_islands.find(island);
            const auto& op         = gr.metadata(nh).get<Op>();

            if (island_backend_it != backends_of_islands.end())
            {
                // Check that backend of the operation coincides with the backend of the island
                // Backend of the island is determined by the backend of the first operation from this island
                if (island_backend_it->second != op.backend)
                {
                    util::throw_error(std::logic_error(island + " contains kernels " + op.k.name +
                                                       " with different backend"));
                }
            }
            else
            {
                backends_of_islands.emplace(island, op.backend);
            }
        }
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `compiler/passes/passes.hpp`
- `stack`
- `ade/util/chain_range.hpp`
- `sstream`
- `precomp.hpp`
- `ade/graph.hpp`
- `compiler/gmodel.hpp`

**Python Imports:**
- `a`
- `Tagged`
- `this`
- `Stack`


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

