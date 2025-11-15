# Documentation for `cmake/checks/OpenCVDetectCudaArch.cu`

## File Metadata

- **Full Path**: `cmake/checks/OpenCVDetectCudaArch.cu`
- **File Name**: `OpenCVDetectCudaArch.cu`
- **File Size**: 725 bytes
- **File Type**: .cu
- **Link to Source**: [cmake/checks/OpenCVDetectCudaArch.cu](../../cmake/checks/OpenCVDetectCudaArch.cu)

## Purpose and Role

This file is located in the `cmake/checks` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#include <iostream>
#include <sstream>
#include <list>

int main()
{
    std::ostringstream arch;
    std::list<std::string> archs;

    int count = 0;
    if (cudaSuccess != cudaGetDeviceCount(&count)){ return -1; }
    if (count == 0) { return -1; }
    for (int device = 0; device < count; ++device)
    {
        cudaDeviceProp prop;
        if (cudaSuccess != cudaGetDeviceProperties(&prop, device)){ continue; }
        arch << prop.major << "." << prop.minor;
        archs.push_back(arch.str());
        arch.str("");
    }
    archs.unique(); // Some devices might have the same arch
    for (std::list<std::string>::iterator it=archs.begin(); it!=archs.end(); ++it)
        std::cout << *it << " ";
    return 0;
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

