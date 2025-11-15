# Documentation for `modules/gapi/include/opencv2/gapi/garray.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/garray.hpp`
- **File Name**: `garray.hpp`
- **File Size**: 15,832 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/garray.hpp](../../../../../modules/gapi/include/opencv2/gapi/garray.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#ifndef OPENCV_GAPI_GARRAY_HPP
#define OPENCV_GAPI_GARRAY_HPP

#include <functional>
#include <ostream>
#include <vector>
#include <memory>

#include <opencv2/gapi/own/exports.hpp>
#include <opencv2/gapi/opencv_includes.hpp>

#include <opencv2/gapi/util/variant.hpp>
#include <opencv2/gapi/util/throw.hpp>
#include <opencv2/gapi/own/assert.hpp>

#include <opencv2/gapi/gmat.hpp>    // flatten_g only!
#include <opencv2/gapi/gscalar.hpp> // flatten_g only!

namespace cv
{
// Forward declaration; GNode and GOrigin are an internal
// (user-inaccessible) classes.
class GNode;
struct GOrigin;
template<typename T> class GArray;

/**
 * \addtogroup gapi_meta_args
 * @{
 */
struct GAPI_EXPORTS_W_SIMPLE GArrayDesc
{
    // FIXME: Body
    // FIXME: Also implement proper operator== then
    bool operator== (const GArrayDesc&) const { return true; }
};
template<typename U> GArrayDesc descr_of(const std::vector<U> &) { return {};}
GAPI_EXPORTS_W inline GArrayDesc empty_array_desc() {return {}; }
/** @} */

std::ostream& operator<<(std::ostream& os, const cv::GArrayDesc &desc);

namespace detail
{
    // ConstructVec is a callback which stores information about T and is used by
    // G-API runtime to construct arrays in host memory (T remains opaque for G-API).
    // ConstructVec is carried into G-API internals by GArrayU.
    // Currently it is suitable for Host (CPU) plugins only, real offload may require
    // more information for manual memory allocation on-device.
    class VectorRef;
    using ConstructVec = std::function<void(VectorRef&)>;

    // This is the base struct for GArrayU type holder
    struct TypeHintBase{virtual ~TypeHintBase() = default;};

    // This class holds type of initial GArray to be checked from GArrayU
    template <typename T>
    struct TypeHint final : public TypeHintBase{};

    // This class strips type information from GArray<T> and makes it usable
    // in the G-API graph compiler (expression unrolling, graph generation, etc).
    // Part of GProtoArg.
    class GAPI_EXPORTS GArrayU
    {
    public:
        GArrayU(const GNode &n, std::size_t out); // Operation result constructor

        template <typename T>
        bool holds() const;                       // Check if was created from GArray<T>

        GOrigin& priv();                          // Internal use only
        const GOrigin& priv() const;              // Internal use only

    protected:
        GArrayU();                                // Default constructor
        GArrayU(const detail::VectorRef& vref);   // Constant value constructor
        template<class> friend class cv::GArray;  //  (available to GArray<T> only)

        void setConstructFcn(ConstructVec &&cv);  // Store T-aware constructor

        template <typename T>
        void specifyType();                       // Store type of initial GArray<T>

        template <typename T>
        void storeKind();

        void setKind(cv::detail::OpaqueKind);

        std::shared_ptr<GOrigin> m_priv;
        std::shared_ptr<TypeHintBase> m_hint;
    };

    template <typename T>
    bool GArrayU::holds() const{
        GAPI_Assert(m_hint != nullptr);
        using U = typename std::decay<T>::type;
        return dynamic_cast<TypeHint<U>*>(m_hint.get()) != nullptr;
    }

    template <typename T>
    void GArrayU::specifyType(){
        m_hint.reset(new TypeHint<typename std::decay<T>::type>);
    }

    template <typename T>
    void GArrayU::storeKind(){
        setKind(cv::detail::GOpaqueTraits<T>::kind);
    }

    // This class represents a typed STL vector reference.
    // Depending on origins, this reference may be either "just a" reference to
    // an object created externally, OR actually own the underlying object
    // (be value holder).
    class BasicVectorRef
    {
    public:
        // These fields are set by the derived class(es)
        std::size_t    m_elemSize = 0ul;
        cv::GArrayDesc m_desc;
        virtual ~BasicVectorRef() {}

        virtual void mov(BasicVectorRef &ref) = 0;
        virtual const void* ptr() const = 0;
        virtual std::size_t size() const = 0;
    };

    template<typename T> class VectorRefT final: public BasicVectorRef
    {
        using empty_t  = util::monostate;
        using ro_ext_t = const std::vector<T> *;
        using rw_ext_t =       std::vector<T> *;
        using rw_own_t =       std::vector<T>  ;
        util::variant<empty_t, ro_ext_t, rw_ext_t, rw_own_t> m_ref;

        inline bool isEmpty() const { return util::holds_alternative<empty_t>(m_ref);  }
        inline bool isROExt() const { return util::holds_alternative<ro_ext_t>(m_ref); }
        inline bool isRWExt() const { return util::holds_alternative<rw_ext_t>(m_ref); }
        inline bool isRWOwn() const { return util::holds_alternative<rw_own_t>(m_ref); }

        void init(const std::vector<T>* vec = nullptr)
        {
            m_elemSize = sizeof(T);
            if (vec) m_desc = cv::descr_of(*vec);
        }

    public:
        VectorRefT() { init(); }
        virtual ~VectorRefT() {}

        explicit VectorRefT(const std::vector<T>& vec) : m_ref(&vec)      { init(&vec); }
        explicit VectorRefT(std::vector<T>& vec)  : m_ref(&vec)           { init(&vec); }
        explicit VectorRefT(std::vector<T>&& vec) : m_ref(std::move(vec)) { init(&vec); }

        // Reset a VectorRefT. Called only for objects instantiated
        // internally in G-API (e.g. temporary GArray<T>'s within a
        // computation).  Reset here means both initialization
        // (creating an object) and reset (discarding its existing
        // content before the next execution).  Must never be called
        // for external VectorRefTs.
        void reset()
        {
            if (isEmpty())
            {
                std::vector<T> empty_vector;
                m_desc = cv::descr_of(empty_vector);
                m_ref  = std::move(empty_vector);
                GAPI_Assert(isRWOwn());
            }
            else if (isRWOwn())
            {
                util::get<rw_own_t>(m_ref).clear();
            }
            else GAPI_Error("InternalError"); // shouldn't be called in *EXT modes
        }

        // Obtain a WRITE reference to underlying object
        // Used by CPU kernel API wrappers when a kernel execution frame
        // is created
        std::vector<T>& wref()
        {
            GAPI_Assert(isRWExt() || isRWOwn());
            if (isRWExt()) return *util::get<rw_ext_t>(m_ref);
            if (isRWOwn()) return  util::get<rw_own_t>(m_ref);
            util::throw_error(std::logic_error("Impossible happened"));
        }

        // Obtain a READ reference to underlying object
        // Used by CPU kernel API wrappers when a kernel execution frame
        // is created
        const std::vector<T>& rref() const
        {
            // ANY vector can be accessed for reading, even if it declared for
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

        virtual void mov(BasicVectorRef &v) override {
            VectorRefT<T> *tv = dynamic_cast<VectorRefT<T>*>(&v);
            GAPI_Assert(tv != nullptr);
            wref() = std::move(tv->wref());
        }

        virtual const void* ptr() const override { return &rref(); }
        virtual std::size_t size() const override { return rref().size(); }
    };

    // This class strips type information from VectorRefT<> and makes it usable
    // in the G-API executables (carrying run-time data/information to kernels).
    // Part of GRunArg.
    // Its methods are typed proxies to VectorRefT<T>.
    // VectorRef maintains "reference" semantics so two copies of VectoRef refer
    // to the same underlying object.
    // FIXME: Put a good explanation on why cv::OutputArray doesn't fit this role
    class VectorRef
    {
        std::shared_ptr<BasicVectorRef> m_ref;
        cv::detail::OpaqueKind m_kind = cv::detail::OpaqueKind::CV_UNKNOWN;

        template<typename T> inline void check() const
        {
            GAPI_DbgAssert(dynamic_cast<VectorRefT<T>*>(m_ref.get()) != nullptr);
            GAPI_Assert(sizeof(T) == m_ref->m_elemSize);
        }

    public:
        VectorRef() = default;
        template<typename T> explicit VectorRef(const std::vector<T>& vec)
            : m_ref(new VectorRefT<T>(vec))
            , m_kind(GOpaqueTraits<T>::kind)
        {}
        template<typename T> explicit VectorRef(std::vector<T>& vec)
            : m_ref(new VectorRefT<T>(vec))
            , m_kind(GOpaqueTraits<T>::kind)
        {}
        template<typename T> explicit VectorRef(std::vector<T>&& vec)
            : m_ref(new VectorRefT<T>(std::move(vec)))
            , m_kind(GOpaqueTraits<T>::kind)
        {}

        cv::detail::OpaqueKind getKind() const
        {
            return m_kind;
        }

        template<typename T> void reset()
        {
            if (!m_ref) m_ref.reset(new VectorRefT<T>());
            check<T>();
            storeKind<T>();
            static_cast<VectorRefT<T>&>(*m_ref).reset();
        }

        template <typename T>
        void storeKind()
        {
            m_kind = cv::detail::GOpaqueTraits<T>::kind;
        }

        template<typename T> std::vector<T>& wref()
        {
            check<T>();
            return static_cast<VectorRefT<T>&>(*m_ref).wref();
        }

        template<typename T> const std::vector<T>& rref() const
        {
            check<T>();
            return static_cast<VectorRefT<T>&>(*m_ref).rref();
        }

        // Check if was created for/from std::vector<T>
        template <typename T> bool holds() const
        {
            if (!m_ref) return false;
            using U = typename std::decay<T>::type;
            return dynamic_cast<VectorRefT<U>*>(m_ref.get()) != nullptr;
        }

        void mov(VectorRef &v)
        {
            m_ref->mov(*v.m_ref);
        }

        cv::GArrayDesc descr_of() const
        {
            return m_ref->m_desc;
        }

        std::size_t size() const
        {
            return m_ref->size();
        }

        // May be used to uniquely identify this object internally
        const void *ptr() const { return m_ref->ptr(); }
    };

    // Helper (FIXME: work-around?)
    // stripping G types to their host types
    // like cv::GArray<GMat> would still map to std::vector<cv::Mat>
    // but not to std::vector<cv::GMat>
#if defined(GAPI_STANDALONE)
#  define FLATTEN_NS cv::gapi::own
#else
#  define FLATTEN_NS cv
#endif
    template<class T> struct flatten_g;
    template<> struct flatten_g<cv::GMat>         { using type = FLATTEN_NS::Mat; };
    template<> struct flatten_g<cv::GScalar>      { using type = FLATTEN_NS::Scalar; };
    template<class T> struct flatten_g<GArray<T>> { using type = std::vector<T>; };
    template<class T> struct flatten_g            { using type = T; };
#undef FLATTEN_NS
    // FIXME: the above mainly duplicates "ProtoToParam" thing from gtyped.hpp
    // but I decided not to include gtyped here - probably worth moving that stuff
    // to some common place? (DM)
} // namespace detail

/** \addtogroup gapi_data_objects
 * @{
 */
/**
 * @brief `cv::GArray<T>` template class represents a list of objects
 * of class `T` in the graph.
 *
 * `cv::GArray<T>` describes a functional relationship between
 * operations consuming and producing arrays of objects of class
 * `T`. The primary purpose of `cv::GArray<T>` is to represent a
 * dynamic list of objects -- where the size of the list is not known
 * at the graph construction or compile time. Examples include: corner
 * and feature detectors (`cv::GArray<cv::Point>`), object detection
 * and tracking  results (`cv::GArray<cv::Rect>`). Programmers can use
 * their own types with `cv::GArray<T>` in the custom operations.
 *
 * Similar to `cv::GScalar`, `cv::GArray<T>` may be value-initialized
 * -- in this case a graph-constant value is associated with the object.
 *
 * `GArray<T>` is a virtual counterpart of `std::vector<T>`, which is
 * usually used to represent the `GArray<T>` data in G-API during the
 * execution.
 *
 * @sa `cv::GOpaque<T>`
 */
template<typename T> class GArray
{
public:
    // Host type (or Flat type) - the type this GArray is actually
    // specified to.
    /// @private
    using HT = typename detail::flatten_g<typename std::decay<T>::type>::type;

    /**
     * @brief Constructs a value-initialized `cv::GArray<T>`
     *
     * `cv::GArray<T>` objects  may have their values
     * be associated at graph construction time. It is useful when
     * some operation has a `cv::GArray<T>` input which doesn't change during
     * the program execution, and is set only once. In this case,
     * there is no need to declare such `cv::GArray<T>` as a graph input.
     *
     * @note The value of `cv::GArray<T>` may be overwritten by assigning some
     * other `cv::GArray<T>` to the object using `operator=` -- on the
     * assignment, the old association or value is discarded.
     *
     * @param v a std::vector<T> to associate with this
     * `cv::GArray<T>` object. Vector data is copied into the
     * `cv::GArray<T>` (no reference to the passed data is held).
     */
    explicit GArray(const std::vector<HT>& v) // Constant value constructor
        : m_ref(detail::GArrayU(detail::VectorRef(v))) { putDetails(); }

    /**
     * @overload
     * @brief Constructs a value-initialized `cv::GArray<T>`
     *
     * @param v a std::vector<T> to associate with this
     * `cv::GArray<T>` object. Vector data is moved into the `cv::GArray<T>`.
     */
    explicit GArray(std::vector<HT>&& v)      // Move-constructor
        : m_ref(detail::GArrayU(detail::VectorRef(std::move(v)))) { putDetails(); }

    /**
     * @brief Constructs an empty `cv::GArray<T>`
     *
     * Normally, empty G-API data objects denote a starting point of
     * the graph. When an empty `cv::GArray<T>` is assigned to a result
     * of some operation, it obtains a functional link to this
     * operation (and is not empty anymore).
     */
    GArray() { putDetails(); }                // Empty constructor

    /// @private
    explicit GArray(detail::GArrayU &&ref)    // GArrayU-based constructor
        : m_ref(ref) { putDetails(); }        //   (used by GCall, not for users)

    /// @private
    detail::GArrayU strip() const {
        return m_ref;
    }
    /// @private
    static void VCtor(detail::VectorRef& vref) {
        vref.reset<HT>();
    }

private:
    void putDetails() {
        m_ref.setConstructFcn(&VCtor);
        m_ref.specifyType<HT>();  // FIXME: to unify those 2 to avoid excessive dynamic_cast
        m_ref.storeKind<HT>();    //
    }

    detail::GArrayU m_ref;
};

/** @} */

} // namespace cv

#endif // OPENCV_GAPI_GARRAY_HPP
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

- **GArray**: A class/struct defined in this file
- **TypeHintBase**: A class/struct defined in this file
- **GNode**: A class/struct defined in this file
- **holds**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **represents**: A class/struct defined in this file
- **strips**: A class/struct defined in this file
- **VectorRefT**: A class/struct defined in this file
- **cv**: A class/struct defined in this file
- **GOrigin**: A class/struct defined in this file
- **BasicVectorRef**: A class/struct defined in this file
- **VectorRef**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **flatten_g**: A class/struct defined in this file
- **arrays**: A class/struct defined in this file
- **GAPI_EXPORTS_W_SIMPLE**: A class/struct defined in this file
- **TypeHint**: A class/struct defined in this file

### Functions and Methods

- **FLATTEN_NS()**: A function/method defined in this file
- **OPENCV_GAPI_GARRAY_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/util/variant.hpp`
- `functional`
- `opencv2/gapi/opencv_includes.hpp`
- `opencv2/gapi/own/exports.hpp`
- `vector`
- `opencv2/gapi/gmat.hpp`
- `memory`
- `opencv2/gapi/own/assert.hpp`
- `opencv2/gapi/gscalar.hpp`
- `ostream`
- `opencv2/gapi/util/throw.hpp`

**Python Imports:**
- `GArray`
- `gtyped.hpp`
- `VectorRefT`
- `std`
- `GArrayU`


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

