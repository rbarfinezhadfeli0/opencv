# Documentation for `modules/gapi/src/compiler/passes/perform_substitution.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/passes/perform_substitution.cpp`
- **File Name**: `perform_substitution.cpp`
- **File Size**: 3,703 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/compiler/passes/perform_substitution.cpp](../../../../../modules/gapi/src/compiler/passes/perform_substitution.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler/passes` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation

#include "pattern_matching.hpp"

#include "ade/util/zip_range.hpp"

namespace cv { namespace gimpl {
namespace {
using Graph = GModel::Graph;

template<typename Iterator>
ade::NodeHandle getNh(Iterator it) { return *it; }

template<>
ade::NodeHandle getNh(SubgraphMatch::M::const_iterator it) { return it->second; }

template<typename Container>
void erase(Graph& g, const Container& c)
{
    for (auto first = c.begin(); first != c.end(); ++first) {
        ade::NodeHandle node = getNh(first);
        if (node == nullptr) continue;  // some nodes might already be erased
        g.erase(node);
    }
}
}  // anonymous namespace

void performSubstitution(GModel::Graph& graph,
                         const Protocol& patternP,
                         const Protocol& substituteP,
                         const SubgraphMatch& patternToGraphMatch)
{
    // 1. substitute input nodes
    const auto& patternIns = patternP.in_nhs;
    const auto& substituteIns = substituteP.in_nhs;

    for (auto it : ade::util::zip(ade::util::toRange(patternIns),
                                  ade::util::toRange(substituteIns))) {
        // Note: we don't replace input DATA nodes here, only redirect their output edges
        const auto& patternDataNode = std::get<0>(it);
        const auto& substituteDataNode = std::get<1>(it);
        const auto& graphDataNode = patternToGraphMatch.inputDataNodes.at(patternDataNode);
        GModel::redirectReaders(graph, substituteDataNode, graphDataNode);
    }

    // 2. substitute output nodes
    const auto& patternOuts = patternP.out_nhs;
    const auto& substituteOuts = substituteP.out_nhs;

    for (auto it : ade::util::zip(ade::util::toRange(patternOuts),
                                  ade::util::toRange(substituteOuts))) {
        // Note: we don't replace output DATA nodes here, only redirect their input edges
        const auto& patternDataNode = std::get<0>(it);
        const auto& substituteDataNode = std::get<1>(it);
        const auto& graphDataNode = patternToGraphMatch.outputDataNodes.at(patternDataNode);

        // delete existing edges (otherwise we cannot redirect)
        auto existingEdges = graphDataNode->inEdges();
        // NB: we cannot iterate over node->inEdges() here directly because it gets modified when
        //     edges are erased. Erasing an edge supposes that src/dst nodes will remove
        //     (correspondingly) out/in edge (which is _our edge_). Now, this deleting means
        //     node->inEdges() will also get updated in the process: so, we'd iterate over a
        //     container which changes in this case. Using supplementary std::vector instead:
        std::vector<ade::EdgeHandle> handles(existingEdges.begin(), existingEdges.end());
        for (const auto& e : handles) {
            graph.erase(e);
        }

        GModel::redirectWriter(graph, substituteDataNode, graphDataNode);
    }

    // 3. erase redundant nodes:
    // erase input data nodes of __substitute__
    erase(graph, substituteIns);

    // erase old start OP nodes of __main graph__
    erase(graph, patternToGraphMatch.startOpNodes);

    // erase old internal nodes of __main graph__
    erase(graph, patternToGraphMatch.internalLayers);

    // erase old finish OP nodes of __main graph__
    erase(graph, patternToGraphMatch.finishOpNodes);

    // erase output data nodes of __substitute__
    erase(graph, substituteOuts);
}

}  // namespace gimpl
}  // namespace cv
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
- `ade/util/zip_range.hpp`
- `pattern_matching.hpp`


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

