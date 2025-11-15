# Documentation for `modules/dnn/src/op_cann.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/op_cann.hpp`
- **File Name**: `op_cann.hpp`
- **File Size**: 4,446 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/op_cann.hpp](../../../modules/dnn/src/op_cann.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_OP_CANN_HPP
#define OPENCV_DNN_OP_CANN_HPP

#ifdef HAVE_CANN
#include "acl/acl.h" // acl* functions
#include "graph/graph.h" // ge::Graph; ge::Operator from operator.h
#include "graph/ge_error_codes.h" // GRAPH_SUCCESS, ...

#ifdef CANN_VERSION_BELOW_6_3_ALPHA002
    #include "op_proto/built-in/inc/all_ops.h" // ge::Conv2D, ...
#else
    #include "built-in/op_proto/inc/all_ops.h" // ge::Conv2D, ...
#endif
#include "graph/tensor.h" // ge::Shape, ge::Tensor, ge::TensorDesc
#include "graph/types.h" // DT_FLOAT, ... ; FORMAT_NCHW, ...

#include "ge/ge_api_types.h" // ge::ir_option::SOC_VERSION
#include "ge/ge_ir_build.h" // build graph

// for fork()
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

#endif // HAVE_CANN

#include <vector>

#ifdef HAVE_CANN
#define ACL_CHECK_RET(f) \
{ \
    if (f != ACL_SUCCESS) \
    { \
        CV_LOG_ERROR(NULL, "CANN check failed, ret = " << f); \
        CV_Error(Error::StsError, "CANN check failed"); \
    } \
}
#define ACL_CHECK_GRAPH_RET(f) \
{ \
    if (f != ge::GRAPH_SUCCESS) \
    { \
        CV_LOG_ERROR(NULL, "CANN graph check failed, ret = " << f); \
        CV_Error(Error::StsError, "CANN graph check failed"); \
    } \
}

#endif

namespace cv { namespace dnn {

#ifdef HAVE_CANN

CV__DNN_INLINE_NS_BEGIN

void switchToCannBackend(Net& net);

CV__DNN_INLINE_NS_END

    class CannNet;

    class AclEnvGuard {
    public:
        explicit AclEnvGuard();
        ~AclEnvGuard();
        static std::shared_ptr<AclEnvGuard> GetAclEnv();

    private:
        static std::shared_ptr<AclEnvGuard> global_acl_env_;
        static std::mutex global_acl_env_mutex_;
    };

    class CannConstOp
    {
    public:
        CannConstOp(const uint8_t* data, const int dtype, const std::vector<int>& shape, const std::string& name);
        std::shared_ptr<ge::op::Const> getOp() { return op_; }
        std::shared_ptr<ge::TensorDesc> getTensorDesc() { return desc_; }
    private:
        std::shared_ptr<ge::op::Const> op_;
        std::shared_ptr<ge::TensorDesc> desc_;
    };

    class CannBackendNode : public BackendNode
    {
    public:
        CannBackendNode(const std::shared_ptr<ge::Operator>& op);
        std::shared_ptr<ge::Operator> getOp();
        std::shared_ptr<CannNet> net;
    private:
        std::shared_ptr<ge::Operator> op_;
    };

    class CannBackendWrapper : public BackendWrapper
    {
    public:
        CannBackendWrapper(const Mat& m);
        ~CannBackendWrapper() { }

        std::shared_ptr<ge::TensorDesc> getTensorDesc() { return desc_; }

        virtual void copyToHost() CV_OVERRIDE;

        virtual void setHostDirty() CV_OVERRIDE;

        Mat* host;
        std::shared_ptr<ge::TensorDesc> desc_;
        std::string name;
    };

    class CannNet
    {
    public:
        explicit CannNet(int deviceId = 0)
            : device_id(deviceId)
        {
            init();
            acl_env = AclEnvGuard::GetAclEnv();
        }
        ~CannNet(); // release private members

        bool empty() const;

        void loadModelBuffer(std::shared_ptr<ge::ModelBufferData> modelBuffer);

        void bindInputWrappers(const std::vector<Ptr<BackendWrapper>>& inputWrappers);
        void bindOutputWrappers(const std::vector<Ptr<BackendWrapper>>& outputWrappers);

        void forward();

        size_t getInputNum() const;
        size_t getOutputNum() const;

    private:
        void init();

        void loadToDevice(); // call aclInit before this API is called
        void createInputDataset();
        void createOutputDataset();

        int getOutputIndexByName(const std::string& name);

        void destroyDataset(aclmdlDataset** dataset);

        std::shared_ptr<AclEnvGuard> acl_env;

        std::vector<Ptr<CannBackendWrapper>> input_wrappers;
        std::vector<Ptr<CannBackendWrapper>> output_wrappers;

        uint32_t model_id{0};
        aclmdlDesc* model_desc{nullptr};
        std::vector<uint8_t> model;
        aclmdlDataset* inputs{nullptr};
        aclmdlDataset* outputs{nullptr};

        int device_id{0};
        aclrtContext context{nullptr};
    };

#endif // HAVE_CANN

}} // namespace cv::dnn

#endif // OPENCV_DNN_OP_CANN_HPP
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

- **CannBackendWrapper**: A class/struct defined in this file
- **AclEnvGuard**: A class/struct defined in this file
- **CannConstOp**: A class/struct defined in this file
- **CannNet**: A class/struct defined in this file
- **CannBackendNode**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_OP_CANN_HPP()**: A function/method defined in this file
- **CANN_VERSION_BELOW_6_3_ALPHA002()**: A function/method defined in this file
- **HAVE_CANN()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `graph/ge_error_codes.h`
- `graph/types.h`
- `sys/wait.h`
- `op_proto/built-in/inc/all_ops.h`
- `acl/acl.h`
- `unistd.h`
- `ge/ge_ir_build.h`
- `stdlib.h`
- `vector`
- `graph/graph.h`
- `ge/ge_api_types.h`
- `sys/mman.h`
- `sys/types.h`
- `built-in/op_proto/inc/all_ops.h`
- `graph/tensor.h`

**Python Imports:**
- `operator.h`


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

