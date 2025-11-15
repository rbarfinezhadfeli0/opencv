# Documentation for `modules/gapi/src/compiler/transactions.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/transactions.hpp`
- **File Name**: `transactions.hpp`
- **File Size**: 5,759 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/compiler/transactions.hpp](../../../../modules/gapi/src/compiler/transactions.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#ifndef OPENCV_GAPI_COMPILER_TRANSACTIONS_HPP
#define OPENCV_GAPI_COMPILER_TRANSACTIONS_HPP

#include <algorithm> // find_if
#include <functional>
#include <list>

#include <ade/graph.hpp>

#include "opencv2/gapi/util/util.hpp" // Seq
#include "opencv2/gapi/own/assert.hpp"

enum class Direction: int {Invalid, In, Out};

////////////////////////////////////////////////////////////////////////////
////
// TODO: Probably it can be moved to ADE
template<class H, class... Metatypes>
class Preserved
{
    using S = typename cv::detail::MkSeq<sizeof...(Metatypes)>::type;
    std::tuple<cv::util::optional<Metatypes>...> m_data;

    template<class T>
    cv::util::optional<T> get(ade::ConstTypedGraph<Metatypes...> g, H h) {
        return g.metadata(h).template contains<T>()
            ? cv::util::make_optional(g.metadata(h).template get<T>())
            : cv::util::optional<T>{};
    }
    template<std::size_t Id>
    int set(ade::TypedGraph<Metatypes...> &g, H &h) {
        const auto &opt = std::get<Id>(m_data);
        if (opt.has_value())
            g.metadata(h).set(opt.value());
        return 0;
    }
    template<int... IIs>
    void copyTo_impl(ade::TypedGraph<Metatypes...> &g, H h, cv::detail::Seq<IIs...>) {
        int unused[] = {0, set<IIs>(g, h)...};
        (void) unused;
    }
public:
    Preserved(const ade::Graph &g, H h) {
        ade::ConstTypedGraph<Metatypes...> tg(g);
        m_data = std::make_tuple(get<Metatypes>(tg, h)...);
    }
    void copyTo(ade::Graph &g, H h) {
        ade::TypedGraph<Metatypes...> tg(g);
        copyTo_impl(tg, h, S{});
    }
};
// Do nothing if there's no metadata
template<class H>
class Preserved<H> {
public:
    Preserved(const ade::Graph &, H) {}
    void copyTo(ade::Graph &, H) {}
};

template<class... Metatypes>
struct ChangeT
{
    struct Base
    {
        virtual void commit  (ade::Graph & ) {}
        virtual void rollback(ade::Graph & ) {}
        virtual ~Base() = default;
    };

    template<typename H> using Preserved = ::Preserved<H, Metatypes...>;

    class NodeCreated final: public Base
    {
        ade::NodeHandle m_node;
    public:
        explicit NodeCreated(const ade::NodeHandle &nh) : m_node(nh) {}
        virtual void rollback(ade::Graph &g) override { g.erase(m_node); }
    };

    // FIXME: maybe extend ADE to clone/copy the whole metadata?
    class DropLink final: public Base
    {
        ade::NodeHandle m_node;
        Direction       m_dir;

        ade::NodeHandle m_sibling;

        Preserved<ade::EdgeHandle> m_meta;

    public:
        DropLink(ade::Graph &g,
                 const ade::NodeHandle &node,
                 const ade::EdgeHandle &edge)
            : m_node(node)
            , m_dir(node == edge->srcNode() ? Direction::Out : Direction::In)
            , m_meta(g, edge)
        {
            m_sibling = (m_dir == Direction::In
                         ? edge->srcNode()
                         : edge->dstNode());
            g.erase(edge);
        }

        virtual void rollback(ade::Graph &g) override
        {
            // FIXME: Need to preserve metadata here!
            // GIslandModel edges now have metadata
            ade::EdgeHandle eh;
            switch(m_dir)
            {
            case Direction::In:  eh = g.link(m_sibling, m_node); break;
            case Direction::Out: eh = g.link(m_node, m_sibling); break;
            default: GAPI_Error("InternalError");
            }
            GAPI_Assert(eh != nullptr);
            m_meta.copyTo(g, eh);
        }
    };

    class NewLink final: public Base
    {
        ade::EdgeHandle m_edge;

    public:
        NewLink(ade::Graph &g,
                const ade::NodeHandle &prod,
                const ade::NodeHandle &cons,
                const ade::EdgeHandle &copy_from = ade::EdgeHandle())
            : m_edge(g.link(prod, cons))
        {
            if (copy_from != nullptr)
            {
                Preserved<ade::EdgeHandle>(g, copy_from).copyTo(g, m_edge);
            }
        }

        virtual void rollback(ade::Graph &g) override
        {
            g.erase(m_edge);
        }
    };

    class DropNode final: public Base
    {
        ade::NodeHandle m_node;

    public:
        explicit DropNode(const ade::NodeHandle &nh)
            : m_node(nh)
        {
            // According to the semantic, node should be disconnected
            // manually before it is dropped
            GAPI_Assert(m_node->inEdges().size()  == 0);
            GAPI_Assert(m_node->outEdges().size() == 0);
        }

        virtual void commit(ade::Graph &g) override
        {
            g.erase(m_node);
        }
    };

    class List
    {
        std::list< std::unique_ptr<Base> > m_changes;

    public:
        template<typename T, typename ...Args>
        void enqueue(Args&&... args)
        {
            std::unique_ptr<Base> p(new T(args...));
            m_changes.push_back(std::move(p));
        }

        void commit(ade::Graph &g)
        {
            // Commit changes in the forward order
            for (auto& ch : m_changes) ch->commit(g);
        }

        void rollback(ade::Graph &g)
        {
            // Rollback changes in the reverse order
            for (auto it = m_changes.rbegin(); it != m_changes.rend(); ++it)
            {
                (*it)->rollback(g);
            }
        }
    };
}; // struct Change
////////////////////////////////////////////////////////////////////////////

#endif // OPENCV_GAPI_COMPILER_TRANSACTIONS_HPP
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

- **NodeCreated**: A class/struct defined in this file
- **DropLink**: A class/struct defined in this file
- **Change**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **ChangeT**: A class/struct defined in this file
- **Base**: A class/struct defined in this file
- **DropNode**: A class/struct defined in this file
- **Direction**: A class/struct defined in this file
- **NewLink**: A class/struct defined in this file
- **List**: A class/struct defined in this file
- **H**: A class/struct defined in this file
- **Preserved**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_COMPILER_TRANSACTIONS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `list`
- `opencv2/gapi/util/util.hpp`
- `functional`
- `opencv2/gapi/own/assert.hpp`
- `algorithm`
- `ade/graph.hpp`


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

