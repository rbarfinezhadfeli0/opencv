# Documentation for `modules/gapi/include/opencv2/gapi/gopaque.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gopaque.hpp`
- **File Name**: `gopaque.hpp`
- **File Size**: 12,382 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gopaque.hpp](../../../../../modules/gapi/include/opencv2/gapi/gopaque.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019-2020 Intel Corporation


#ifndef OPENCV_GAPI_GOPAQUE_HPP
#define OPENCV_GAPI_GOPAQUE_HPP

#include <functional>
#include <ostream>
#include <memory>

#include <opencv2/gapi/own/exports.hpp>
#include <opencv2/gapi/opencv_includes.hpp>

#include <opencv2/gapi/util/any.hpp>
#include <opencv2/gapi/util/variant.hpp>
#include <opencv2/gapi/util/throw.hpp>
#include <opencv2/gapi/util/type_traits.hpp>
#include <opencv2/gapi/own/assert.hpp>

#include <opencv2/gapi/gcommon.hpp>  // OpaqueKind
#include <opencv2/gapi/garray.hpp>  // TypeHintBase

namespace cv
{
// Forward declaration; GNode and GOrigin are an internal
// (user-inaccessible) classes.
class GNode;
struct GOrigin;
template<typename T> class GOpaque;

/**
 * \addtogroup gapi_meta_args
 * @{
 */
struct GAPI_EXPORTS_W_SIMPLE GOpaqueDesc
{
    // FIXME: Body
    // FIXME: Also implement proper operator== then
    bool operator== (const GOpaqueDesc&) const { return true; }
};
template<typename U> GOpaqueDesc descr_of(const U &) { return {};}
GAPI_EXPORTS_W inline GOpaqueDesc empty_gopaque_desc() {return {}; }
/** @} */

std::ostream& operator<<(std::ostream& os, const cv::GOpaqueDesc &desc);

namespace detail
{
    // ConstructOpaque is a callback which stores information about T and is used by
    // G-API runtime to construct an object in host memory (T remains opaque for G-API).
    // ConstructOpaque is carried into G-API internals by GOpaqueU.
    // Currently it is suitable for Host (CPU) plugins only, real offload may require
    // more information for manual memory allocation on-device.
    class OpaqueRef;
    using ConstructOpaque = std::function<void(OpaqueRef&)>;

    // FIXME: garray.hpp already contains hint classes (for actual T type verification),
    // need to think where it can be moved (currently opaque uses it from garray)

    // This class strips type information from GOpaque<T> and makes it usable
    // in the G-API graph compiler (expression unrolling, graph generation, etc).
    // Part of GProtoArg.
    class GAPI_EXPORTS GOpaqueU
    {
    public:
        GOpaqueU(const GNode &n, std::size_t out); // Operation result constructor

        template <typename T>
        bool holds() const;                       // Check if was created from GOpaque<T>

        GOrigin& priv();                          // Internal use only
        const GOrigin& priv() const;              // Internal use only

    protected:
        GOpaqueU();                                // Default constructor
        template<class> friend class cv::GOpaque;  // (available for GOpaque<T> only)

        void setConstructFcn(ConstructOpaque &&cv);  // Store T-aware constructor

        template <typename T>
        void specifyType();                       // Store type of initial GOpaque<T>

        template <typename T>
        void storeKind();

        void setKind(cv::detail::OpaqueKind);

        std::shared_ptr<GOrigin> m_priv;
        std::shared_ptr<TypeHintBase> m_hint;
    };

    template <typename T>
    bool GOpaqueU::holds() const{
        GAPI_Assert(m_hint != nullptr);
        using U = util::decay_t<T>;
        return dynamic_cast<TypeHint<U>*>(m_hint.get()) != nullptr;
    }

    template <typename T>
    void GOpaqueU::specifyType(){
        m_hint.reset(new TypeHint<util::decay_t<T>>);
    }

    template <typename T>
    void GOpaqueU::storeKind(){
        // FIXME: Add assert here on cv::Mat and cv::Scalar?
        setKind(cv::detail::GOpaqueTraits<T>::kind);
    }

    // This class represents a typed object reference.
    // Depending on origins, this reference may be either "just a" reference to
    // an object created externally, OR actually own the underlying object
    // (be value holder).
    class BasicOpaqueRef
    {
    public:
        cv::GOpaqueDesc m_desc;
        virtual ~BasicOpaqueRef() {}

        virtual void mov(BasicOpaqueRef &ref) = 0;
        virtual const void* ptr() const = 0;
        virtual void set(const cv::util::any &a) = 0;
    };

    template<typename T> class OpaqueRefT final: public BasicOpaqueRef
    {
        using empty_t  = util::monostate;
        using ro_ext_t = const T *;
        using rw_ext_t =       T *;
        using rw_own_t =       T  ;
        util::variant<empty_t, ro_ext_t, rw_ext_t, rw_own_t> m_ref;

        inline bool isEmpty() const { return util::holds_alternative<empty_t>(m_ref);  }
        inline bool isROExt() const { return util::holds_alternative<ro_ext_t>(m_ref); }
        inline bool isRWExt() const { return util::holds_alternative<rw_ext_t>(m_ref); }
        inline bool isRWOwn() const { return util::holds_alternative<rw_own_t>(m_ref); }

        void init(const T* obj = nullptr)
        {
            if (obj) m_desc = cv::descr_of(*obj);
        }

    public:
        OpaqueRefT() { init(); }
        virtual ~OpaqueRefT() {}

        explicit OpaqueRefT(const T&  obj) : m_ref(&obj)           { init(&obj); }
        explicit OpaqueRefT(      T&  obj) : m_ref(&obj)           { init(&obj); }
        explicit OpaqueRefT(      T&& obj) : m_ref(std::move(obj)) { init(&obj); }

        // Reset a OpaqueRefT. Called only for objects instantiated
        // internally in G-API (e.g. temporary GOpaque<T>'s within a
        // computation).  Reset here means both initialization
        // (creating an object) and reset (discarding its existing
        // content before the next execution). Must never be called
        // for external OpaqueRefTs.
        void reset()
        {
            if (isEmpty())
            {
                T empty_obj{};
                m_desc = cv::descr_of(empty_obj);
                m_ref  = std::move(empty_obj);
                GAPI_Assert(isRWOwn());
            }
            else if (isRWOwn())
            {
                util::get<rw_own_t>(m_ref) = {};
            }
            else GAPI_Error("InternalError"); // shouldn't be called in *EXT modes
        }

        // Obtain a WRITE reference to underlying object
        // Used by CPU kernel API wrappers when a kernel execution frame
        // is created
        T& wref()
        {
            GAPI_Assert(isRWExt() || isRWOwn());
            if (isRWExt()) return *util::get<rw_ext_t>(m_ref);
            if (isRWOwn()) return  util::get<rw_own_t>(m_ref);
            util::throw_error(std::logic_error("Impossible happened"));
        }

        // Obtain a READ reference to underlying object
        // Used by CPU kernel API wrappers when a kernel execution frame
        // is created
        const T& rref() const
        {
            // ANY object can be accessed for reading, even if it declared for
            // output. Example -- a GComputation from [in] to [out1,out2]
            // where [out2] is a result of operation applied to [out1]:
            //
            //            GComputation boundary
            //            . . . . . . .
            //            .           .
            //     [in] ----> foo() ----> [out1]
            //            .           .    :
            //            .           . . .:. . .
            //            .                V    .
            //            .              bar() ---> [out2]
            //            . . . . . . . . . . . .
            //
            if (isROExt()) return *util::get<ro_ext_t>(m_ref);
            if (isRWExt()) return *util::get<rw_ext_t>(m_ref);
            if (isRWOwn()) return  util::get<rw_own_t>(m_ref);
            util::throw_error(std::logic_error("Impossible happened"));
        }

        virtual void mov(BasicOpaqueRef &v) override {
            OpaqueRefT<T> *tv = dynamic_cast<OpaqueRefT<T>*>(&v);
            GAPI_Assert(tv != nullptr);
            wref() = std::move(tv->wref());
        }

        virtual const void* ptr() const override { return &rref(); }

        virtual void set(const cv::util::any &a) override {
            wref() = util::any_cast<T>(a);
        }
    };

    // This class strips type information from OpaqueRefT<> and makes it usable
    // in the G-API executables (carrying run-time data/information to kernels).
    // Part of GRunArg.
    // Its methods are typed proxies to OpaqueRefT<T>.
    // OpaqueRef maintains "reference" semantics so two copies of OpaqueRef refer
    // to the same underlying object.
    class OpaqueRef
    {
        std::shared_ptr<BasicOpaqueRef> m_ref;
        cv::detail::OpaqueKind m_kind = cv::detail::OpaqueKind::CV_UNKNOWN;

        template<typename T> inline void check() const
        {
            GAPI_DbgAssert(dynamic_cast<OpaqueRefT<T>*>(m_ref.get()) != nullptr);
        }

    public:
        OpaqueRef() = default;

        template<
            typename T,
            typename = util::are_different_t<OpaqueRef, T>
        >
        // FIXME: probably won't work with const object
        explicit OpaqueRef(T&& obj) :
            m_ref(new OpaqueRefT<util::decay_t<T>>(std::forward<T>(obj))),
            m_kind(GOpaqueTraits<util::decay_t<T>>::kind) {}

        cv::detail::OpaqueKind getKind() const
        {
            return m_kind;
        }

        template<typename T> void reset()
        {
            if (!m_ref) m_ref.reset(new OpaqueRefT<T>());
            check<T>();
            storeKind<T>();
            static_cast<OpaqueRefT<T>&>(*m_ref).reset();
        }

        template <typename T>
        void storeKind()
        {
            m_kind = cv::detail::GOpaqueTraits<T>::kind;
        }

        template<typename T> T& wref()
        {
            check<T>();
            return static_cast<OpaqueRefT<T>&>(*m_ref).wref();
        }

        template<typename T> const T& rref() const
        {
            check<T>();
            return static_cast<OpaqueRefT<T>&>(*m_ref).rref();
        }

        void mov(OpaqueRef &v)
        {
            m_ref->mov(*v.m_ref);
        }

        cv::GOpaqueDesc descr_of() const
        {
            return m_ref->m_desc;
        }

        // May be used to uniquely identify this object internally
        const void *ptr() const { return m_ref->ptr(); }

        // Introduced for in-graph meta handling
        OpaqueRef& operator= (const cv::util::any &a)
        {
            m_ref->set(a);
            return *this;
        }
    };
} // namespace detail

/** \addtogroup gapi_data_objects
 * @{
 */
/**
 * @brief `cv::GOpaque<T>` template class represents an object of
 * class `T` in the graph.
 *
 * `cv::GOpaque<T>` describes a functional relationship between operations
 * consuming and producing object of class `T`. `cv::GOpaque<T>` is
 * designed to extend G-API with user-defined data types, which are
 * often required with user-defined operations. G-API can't apply any
 * optimizations to user-defined types since these types are opaque to
 * the framework. However, there is a number of G-API operations
 * declared with `cv::GOpaque<T>` as a return type,
 * e.g. cv::gapi::streaming::timestamp() or cv::gapi::streaming::size().
 *
 * @sa `cv::GArray<T>`
 */
template<typename T> class GOpaque
{
public:
    // Host type (or Flat type) - the type this GOpaque is actually
    // specified to.
    /// @private
    using HT = typename detail::flatten_g<util::decay_t<T>>::type;

    /**
     * @brief Constructs an empty `cv::GOpaque<T>`
     *
     * Normally, empty G-API data objects denote a starting point of
     * the graph. When an empty `cv::GOpaque<T>` is assigned to a result
     * of some operation, it obtains a functional link to this
     * operation (and is not empty anymore).
     */
    GOpaque() { putDetails(); }              // Empty constructor

    /// @private
    explicit GOpaque(detail::GOpaqueU &&ref) // GOpaqueU-based constructor
        : m_ref(ref) { putDetails(); }       // (used by GCall, not for users)

    /// @private
    detail::GOpaqueU strip() const {
        return m_ref;
    }
    /// @private
    static void Ctor(detail::OpaqueRef& ref) {
        ref.reset<HT>();
    }
private:
    void putDetails() {
        m_ref.setConstructFcn(&Ctor);
        m_ref.specifyType<HT>();
        m_ref.storeKind<HT>();
    }

    detail::GOpaqueU m_ref;
};

/** @} */

} // namespace cv

#endif // OPENCV_GAPI_GOPAQUE_HPP
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

- **GOpaque**: A class/struct defined in this file
- **GNode**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **represents**: A class/struct defined in this file
- **BasicOpaqueRef**: A class/struct defined in this file
- **strips**: A class/struct defined in this file
- **cv**: A class/struct defined in this file
- **an**: A class/struct defined in this file
- **GOrigin**: A class/struct defined in this file
- **OpaqueRef**: A class/struct defined in this file
- **OpaqueRefT**: A class/struct defined in this file
- **GAPI_EXPORTS_W_SIMPLE**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GOPAQUE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/util/variant.hpp`
- `functional`
- `opencv2/gapi/garray.hpp`
- `opencv2/gapi/opencv_includes.hpp`
- `opencv2/gapi/own/exports.hpp`
- `opencv2/gapi/util/type_traits.hpp`
- `opencv2/gapi/util/any.hpp`
- `memory`
- `opencv2/gapi/own/assert.hpp`
- `opencv2/gapi/gcommon.hpp`
- `ostream`
- `opencv2/gapi/util/throw.hpp`

**Python Imports:**
- `GOpaque`
- `garray`
- `OpaqueRefT`


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

