# Documentation for `modules/dnn/src/halide_scheduler.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/halide_scheduler.cpp`
- **File Name**: `halide_scheduler.cpp`
- **File Size**: 9,517 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/halide_scheduler.cpp](../../../modules/dnn/src/halide_scheduler.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2017, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "precomp.hpp"
#include "halide_scheduler.hpp"
#include "op_halide.hpp"

namespace cv
{
namespace dnn
{

#ifdef HAVE_HALIDE
static void applySplit(const FileNode& directive, Halide::Func& func,
                       const FileNode& params)
{
    for (const auto& varNode : directive)
    {
        const std::string varName = varNode.name();
        const std::string factorName = (std::string)varNode;
        Halide::Var var(varName);
        Halide::Var outerVar(varName + "o");
        Halide::Var innerVar(varName + "i");
        // If split factor is integer or parameters map has parameter value.
        CV_Assert(varNode.isString() && !params[factorName].empty() ||
                  varNode.isInt());
        int factor = (int)(varNode.isInt() ? varNode : params[factorName]);
        func.split(var, outerVar, innerVar, factor);
    }
}

static void applyReorder(const FileNode& directive, Halide::Func& func)
{
    std::string varName;
    const int numVars = directive.size();
    std::vector<Halide::VarOrRVar> reorderedVars;
    reorderedVars.reserve(numVars);
    for (int i = 0; i < numVars; ++i)
    {
        directive[i] >> varName;
        reorderedVars.push_back(Halide::Var(varName));
    }
    func.reorder(reorderedVars);
}

static void applyFuse(const FileNode& directive, Halide::Func& func)
{
    CV_Assert(directive["src"].size() >= 2);
    CV_Assert(directive["dst"].size() == 1);

    std::string str;
    directive["src"][0] >> str;
    Halide::Var firstVar(str);
    directive["src"][1] >> str;
    Halide::Var secondVar(str);
    directive["dst"] >> str;
    Halide::Var dstVar(str);

    func.fuse(firstVar, secondVar, dstVar);
    for (int i = 2, n = directive["src"].size(); i < n; ++i)
    {
        directive["src"][i] >> str;
        func.fuse(Halide::Var(str), dstVar, dstVar);
    }
}

static void applyParallel(const FileNode& directive, Halide::Func& func)
{
    std::string varName;
    if (directive.isSeq())
    {
        for (int i = 0, n = directive.size(); i < n; ++i)
        {
            directive[i] >> varName;
            func.parallel(Halide::Var(varName));
        }
    }
    else
    {
        directive >> varName;
        func.parallel(Halide::Var(varName));
    }
}

static void applyUnroll(const FileNode& directive, Halide::Func& func)
{
    std::string varName;
    if (directive.isSeq())
    {
        for (int i = 0, n = directive.size(); i < n; ++i)
        {
            directive[i] >> varName;
            func.unroll(Halide::Var(varName));
        }
    }
    else
    {
        directive >> varName;
        func.unroll(Halide::Var(varName));
    }
}

static void applyVectorize(const FileNode& directive, Halide::Func& func,
                           const FileNode& params)
{
    for (const auto& varNode : directive)
    {
        const std::string varName = varNode.name();
        const std::string factorName = (std::string)varNode;
        // If split factor is integer or parameters map has parameter value.
        CV_Assert(varNode.isString() && !params[factorName].empty() ||
                  varNode.isInt());
        int factor = (int)(varNode.isInt() ? varNode : params[factorName]);
        Halide::Var var(varName);
        Halide::Var inner(varName + "v");
        func.split(var, var, inner, factor);
        func.vectorize(inner);
    }
}

static void applyStoreAt(const FileNode& directive, Halide::Func& func,
                         std::map<std::string, Halide::Func>& funcsMap)
{
    for (const auto& funcNode : directive)
    {
        const std::string targetFuncName = funcNode.name();
        if (funcsMap.find(targetFuncName) == funcsMap.end())
            CV_Error(cv::Error::StsParseError, "Function " + targetFuncName +
                     " is not represented in Halide pipeline");
        Halide::Func targetFunc = funcsMap[targetFuncName];
        func.store_at(targetFunc, (std::string)funcNode);
        break;
    }
}

static void applyComputeAt(const FileNode& directive, Halide::Func& func,
                           std::map<std::string, Halide::Func>& funcsMap)
{
    for (const auto& funcNode : directive)
    {
        const std::string targetFuncName = funcNode.name();
        if (funcsMap.find(targetFuncName) == funcsMap.end())
            CV_Error(cv::Error::StsParseError, "Function " + targetFuncName +
                     " is not represented in Halide pipeline");
        Halide::Func targetFunc = funcsMap[targetFuncName];
        func.compute_at(targetFunc, (std::string)funcNode);
        break;
    }
}

static void applyComputeRoot(const FileNode& directive, Halide::Func& func)
{
    bool compute_root;
    directive >> compute_root;
    if (compute_root)
        func.compute_root();
}

static void applyGpuBlocks(const FileNode& directive, Halide::Func& func)
{
    std::string varName;
    for (int i = 0, n = directive.size(); i < n; ++i)
    {
        directive[i] >> varName;
        func.gpu_blocks(Halide::Var(varName));
    }
}

static void applyGpuThreads(const FileNode& directive, Halide::Func& func)
{
    std::string varName;
    for (int i = 0, n = directive.size(); i < n; ++i)
    {
        directive[i] >> varName;
        func.gpu_threads(Halide::Var(varName));
    }
}

static void apply(const FileNode& directives, Halide::Func& func,
                  std::map<std::string, Halide::Func>& funcsMap,
                  const FileNode& params)
{
    for (const auto& directive : directives)
    {
        if (directive.name() == "split")
            applySplit(directive, func, params);
        else if (directive.name() == "reorder")
            applyReorder(directive, func);
        else if (directive.name() == "fuse")
            applyFuse(directive, func);
        else if (directive.name() == "parallel")
            applyParallel(directive, func);
        else if (directive.name() == "unroll")
            applyUnroll(directive, func);
        else if (directive.name() == "vectorize")
            applyVectorize(directive, func, params);
        else if (directive.name() == "store_at")
            applyStoreAt(directive, func, funcsMap);
        else if (directive.name() == "compute_at")
            applyComputeAt(directive, func, funcsMap);
        else if (directive.name() == "compute_root")
            applyComputeRoot(directive, func);
        else if (directive.name() == "gpu_blocks")
            applyGpuBlocks(directive, func);
        else if (directive.name() == "gpu_threads")
            applyGpuThreads(directive, func);
        else
            CV_Error(Error::StsNotImplemented, "Scheduling directive " +
                     directive.name() + " is not implemented.");
    }
}

// Remove any numeric symbols after '$' sign.
static std::string Deunique(std::string str)
{
    int pos = -1;
    do
    {
        pos = str.find('$');
        if (pos != -1)
        {
            int len = str.find_first_not_of("0123456789", pos + 1) - pos;
            str = str.replace(pos, len, "");
        }
    }
    while (pos != -1);
    return str;
}
#endif  // HAVE_HALIDE

HalideScheduler::HalideScheduler(const std::string& configFile)
{
    if (!configFile.empty())
        fs = FileStorage(configFile, FileStorage::READ);
}

HalideScheduler::~HalideScheduler()
{
    if (fs.isOpened())
        fs.release();
}

bool HalideScheduler::process(Ptr<BackendNode>& node)
{
#ifdef HAVE_HALIDE
    if (!fs.isOpened())
        return false;

    const FileNode& scheduleNode = fs["scheduling"];
    if (scheduleNode.empty())
        CV_Error(cv::Error::StsParseError, "Scheduling file should has scheduling node");

    std::string str;
    std::map<std::string, Halide::Func> funcsMap;  // Scheduled functions.
    // For every function, from top to bottom, we try to find a scheduling node.
    // Scheduling is successful (return true) if for the first function (top)
    // node is represented.
    CV_Assert(!node.empty());
    std::vector<Halide::Func>& funcs = node.dynamicCast<HalideBackendNode>()->funcs;
    for (int i = funcs.size() - 1; i >= 0; --i)
    {
        Halide::Func& func = funcs[i];
        // For functions with the same name Halide generates unique names
        // for example func, func$1, func$2.
        // They are always formed with '$' and number.
        std::string funcName = Deunique(func.name());

        const FileNode& funcNode = scheduleNode[funcName];
        if (!funcNode.empty())
        {
            if (!funcNode["pattern"].empty())
            {
                funcNode["pattern"] >> str;
                if (fs["patterns"][str].empty())
                    CV_Error(cv::Error::StsParseError, "Scheduling pattern " + str +
                                                       " is not defined");
                apply(fs["patterns"][str], func, funcsMap, funcNode["params"]);
            }
            else
            {
                apply(funcNode, func, funcsMap, funcNode["params"]);
            }
        }
        else
        {
            if (funcsMap.empty())
                return false;
        }
        funcsMap[funcName] = func;
    }
    return true;
#endif  // HAVE_HALIDE
    return false;
}

}  // namespace dnn
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

### Functions and Methods

- **HAVE_HALIDE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `op_halide.hpp`
- `precomp.hpp`
- `halide_scheduler.hpp`

**Python Imports:**
- `top`


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

