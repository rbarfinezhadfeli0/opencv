# Documentation for `modules/dnn/misc/python/pyopencv_dnn.hpp`

## File Metadata

- **Full Path**: `modules/dnn/misc/python/pyopencv_dnn.hpp`
- **File Name**: `pyopencv_dnn.hpp`
- **File Size**: 7,417 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/misc/python/pyopencv_dnn.hpp](../../../../modules/dnn/misc/python/pyopencv_dnn.hpp)

## Purpose and Role

This file is located in the `modules/dnn/misc/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifdef HAVE_OPENCV_DNN
typedef dnn::DictValue LayerId;
typedef std::vector<dnn::MatShape> vector_MatShape;
typedef std::vector<std::vector<dnn::MatShape> > vector_vector_MatShape;

template<>
bool pyopencv_to(PyObject *o, dnn::DictValue &dv, const ArgInfo& info)
{
    CV_UNUSED(info);
    if (!o || o == Py_None)
        return true; //Current state will be used
    else if (PyLong_Check(o))
    {
        dv = dnn::DictValue((int64)PyLong_AsLongLong(o));
        return true;
    }
    else if (PyInt_Check(o))
    {
        dv = dnn::DictValue((int64)PyInt_AS_LONG(o));
        return true;
    }
    else if (PyFloat_Check(o))
    {
        dv = dnn::DictValue(PyFloat_AsDouble(o));
        return true;
    }
    else
    {
        std::string str;
        if (getUnicodeString(o, str))
        {
            dv = dnn::DictValue(str);
            return true;
        }
    }
    return false;
}

template<typename T>
PyObject* pyopencv_from(const dnn::DictValue &dv)
{
    if (dv.size() > 1)
    {
        std::vector<T> vec(dv.size());
        for (int i = 0; i < dv.size(); ++i)
            vec[i] = dv.get<T>(i);
        return pyopencv_from_generic_vec(vec);
    }
    else
        return pyopencv_from(dv.get<T>());
}

template<>
PyObject* pyopencv_from(const dnn::DictValue &dv)
{
    if (dv.isInt()) return pyopencv_from<int>(dv);
    if (dv.isReal()) return pyopencv_from<float>(dv);
    if (dv.isString()) return pyopencv_from<String>(dv);
    CV_Error(Error::StsNotImplemented, "Unknown value type");
    return NULL;
}

template<>
PyObject* pyopencv_from(const dnn::LayerParams& lp)
{
    PyObject* dict = PyDict_New();
    for (std::map<String, dnn::DictValue>::const_iterator it = lp.begin(); it != lp.end(); ++it)
    {
        CV_Assert(!PyDict_SetItemString(dict, it->first.c_str(), pyopencv_from(it->second)));
    }
    return dict;
}

template<>
bool pyopencv_to(PyObject *o, dnn::LayerParams &lp, const ArgInfo& info)
{
    CV_Assert(PyDict_Check(o));
    PyObject *key, *value;
    Py_ssize_t pos = 0;
    std::string keyName;
    while (PyDict_Next(o, &pos, &key, &value)) {
        getUnicodeString(key, keyName);
        dnn::DictValue dv;
        pyopencv_to(value, dv, info);
        lp.set(keyName, dv);
    }
    return true;
}

template<>
PyObject* pyopencv_from(const std::vector<dnn::Target> &t)
{
    return pyopencv_from(std::vector<int>(t.begin(), t.end()));
}

class pycvLayer CV_FINAL : public dnn::Layer
{
public:
    pycvLayer(const dnn::LayerParams &params, PyObject* pyLayer) : Layer(params)
    {
        PyGILState_STATE gstate;
        gstate = PyGILState_Ensure();

        PyObject* args = PyTuple_New(2);
        CV_Assert(!PyTuple_SetItem(args, 0, pyopencv_from(params)));
        CV_Assert(!PyTuple_SetItem(args, 1, pyopencv_from(params.blobs)));
        o = PyObject_CallObject(pyLayer, args);

        Py_DECREF(args);
        PyGILState_Release(gstate);
        if (!o)
            CV_Error(Error::StsError, "Failed to create an instance of custom layer");
    }

    static void registerLayer(const std::string& type, PyObject* o)
    {
        std::map<std::string, std::vector<PyObject*> >::iterator it = pyLayers.find(type);
        if (it != pyLayers.end())
            it->second.push_back(o);
        else
            pyLayers[type] = std::vector<PyObject*>(1, o);
    }

    static void unregisterLayer(const std::string& type)
    {
        std::map<std::string, std::vector<PyObject*> >::iterator it = pyLayers.find(type);
        if (it != pyLayers.end())
        {
            if (it->second.size() > 1)
                it->second.pop_back();
            else
                pyLayers.erase(it);
        }
    }

    static Ptr<dnn::Layer> create(dnn::LayerParams &params)
    {
        std::map<std::string, std::vector<PyObject*> >::iterator it = pyLayers.find(params.type);
        if (it == pyLayers.end())
            CV_Error(Error::StsNotImplemented, "Layer with a type \"" + params.type +
                                               "\" is not implemented");
        CV_Assert(!it->second.empty());
        return Ptr<dnn::Layer>(new pycvLayer(params, it->second.back()));
    }

    virtual bool getMemoryShapes(const std::vector<std::vector<int> > &inputs,
                                 const int,
                                 std::vector<std::vector<int> > &outputs,
                                 std::vector<std::vector<int> > &) const CV_OVERRIDE
    {
        PyGILState_STATE gstate;
        gstate = PyGILState_Ensure();

        PyObject* args = PyList_New(inputs.size());
        for(size_t i = 0; i < inputs.size(); ++i)
            PyList_SetItem(args, i, pyopencv_from_generic_vec(inputs[i]));

        PyObject* res = PyObject_CallMethodObjArgs(o, PyString_FromString("getMemoryShapes"), args, NULL);
        Py_DECREF(args);
        PyGILState_Release(gstate);
        if (!res)
            CV_Error(Error::StsNotImplemented, "Failed to call \"getMemoryShapes\" method");
        CV_Assert(pyopencv_to_generic_vec(res, outputs, ArgInfo("", 0)));
        return false;
    }

    virtual void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays) CV_OVERRIDE
    {
        PyGILState_STATE gstate;
        gstate = PyGILState_Ensure();

        std::vector<Mat> inputs, outputs;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);

        PyObject* args = pyopencv_from(inputs);
        PyObject* res = PyObject_CallMethodObjArgs(o, PyString_FromString("forward"), args, NULL);
        Py_DECREF(args);
        if (!res)
            CV_Error(Error::StsNotImplemented, "Failed to call \"forward\" method");

        std::vector<Mat> pyOutputs;
        CV_Assert(pyopencv_to(res, pyOutputs, ArgInfo("", 0)));
        Py_DECREF(res);
        PyGILState_Release(gstate);

        CV_Assert(pyOutputs.size() == outputs.size());
        for (size_t i = 0; i < outputs.size(); ++i)
        {
            CV_Assert(pyOutputs[i].size == outputs[i].size);
            CV_Assert(pyOutputs[i].type() == outputs[i].type());
            pyOutputs[i].copyTo(outputs[i]);
        }
    }

private:
    // Map layers types to python classes.
    static std::map<std::string, std::vector<PyObject*> > pyLayers;
    PyObject* o;  // Instance of implemented python layer.
};

std::map<std::string, std::vector<PyObject*> > pycvLayer::pyLayers;

static PyObject *pyopencv_cv_dnn_registerLayer(PyObject*, PyObject *args, PyObject *kw)
{
    const char *keywords[] = { "type", "class", NULL };
    char* layerType;
    PyObject *classInstance;

    if (!PyArg_ParseTupleAndKeywords(args, kw, "sO", (char**)keywords, &layerType, &classInstance))
        return NULL;
    if (!PyCallable_Check(classInstance)) {
        PyErr_SetString(PyExc_TypeError, "class must be callable");
        return NULL;
    }

    pycvLayer::registerLayer(layerType, classInstance);
    dnn::LayerFactory::registerLayer(layerType, pycvLayer::create);
    Py_RETURN_NONE;
}

static PyObject *pyopencv_cv_dnn_unregisterLayer(PyObject*, PyObject *args, PyObject *kw)
{
    const char *keywords[] = { "type", NULL };
    char* layerType;

    if (!PyArg_ParseTupleAndKeywords(args, kw, "s", (char**)keywords, &layerType))
        return NULL;

    pycvLayer::unregisterLayer(layerType);
    dnn::LayerFactory::unregisterLayer(layerType);
    Py_RETURN_NONE;
}

#endif  // HAVE_OPENCV_DNN
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

- **must**: A class/struct defined in this file
- **pycvLayer**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **HAVE_OPENCV_DNN()**: A function/method defined in this file
- **dnn()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

