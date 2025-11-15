# Documentation for `docs/platforms/android/build-tests/test_cmake_build.py_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/android/build-tests/test_cmake_build.py_docs.md`
- **File Name**: `test_cmake_build.py_docs.md`
- **File Size**: 9,601 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/android/build-tests/test_cmake_build.py_docs.md](../../../../docs/platforms/android/build-tests/test_cmake_build.py_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/android/build-tests` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/android/build-tests/test_cmake_build.py`

## File Metadata

- **Full Path**: `platforms/android/build-tests/test_cmake_build.py`
- **File Name**: `test_cmake_build.py`
- **File Size**: 5,842 bytes
- **File Type**: .py
- **Link to Source**: [platforms/android/build-tests/test_cmake_build.py](../../../platforms/android/build-tests/test_cmake_build.py)

## Purpose and Role

This file is located in the `platforms/android/build-tests` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

import unittest
import os, sys, subprocess, argparse, shutil, re
import logging as log

log.basicConfig(format='%(message)s', level=log.DEBUG)

CMAKE_TEMPLATE='''\
CMAKE_MINIMUM_REQUIRED(VERSION 3.5)

# Enable C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

SET(PROJECT_NAME hello-android)
PROJECT(${PROJECT_NAME})

FIND_PACKAGE(OpenCV REQUIRED %(libset)s)
FILE(GLOB srcs "*.cpp")

ADD_EXECUTABLE(${PROJECT_NAME} ${srcs})
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${OpenCV_LIBS} dl z)
'''

CPP_TEMPLATE = '''\
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
using namespace cv;
const char* message = "Hello Android!";
int main(int argc, char* argv[])
{
  (void)argc; (void)argv;
  printf("%s\\n", message);
  Size textsize = getTextSize(message, FONT_HERSHEY_COMPLEX, 3, 5, 0);
  Mat img(textsize.height + 20, textsize.width + 20, CV_32FC1, Scalar(230,230,230));
  putText(img, message, Point(10, img.rows - 10), FONT_HERSHEY_COMPLEX, 3, Scalar(0, 0, 0), 5);
  imwrite("/mnt/sdcard/HelloAndroid.png", img);
  return 0;
}
'''

#===================================================================================================

class TestCmakeBuild(unittest.TestCase):
    def __init__(self, libset, abi, cmake_vars, opencv_cmake_path, workdir, *args, **kwargs):
        unittest.TestCase.__init__(self, *args, **kwargs)
        self.libset = libset
        self.abi = abi
        self.cmake_vars = cmake_vars
        self.opencv_cmake_path = opencv_cmake_path
        self.workdir = workdir
        self.srcdir = os.path.join(self.workdir, "src")
        self.bindir = os.path.join(self.workdir, "build")

    def shortDescription(self):
        return "ABI: %s, LIBSET: %s" % (self.abi, self.libset)

    def getCMakeToolchain(self):
        if True:
            toolchain = os.path.join(os.environ['ANDROID_NDK'], 'build', 'cmake', 'android.toolchain.cmake')
            if os.path.exists(toolchain):
                return toolchain
        toolchain = os.path.join(self.opencv_cmake_path, "android.toolchain.cmake")
        if os.path.exists(toolchain):
            return toolchain
        else:
            raise Exception("Can't find toolchain")

    def gen_cmakelists(self):
        return CMAKE_TEMPLATE % {"libset": self.libset}

    def gen_code(self):
        return CPP_TEMPLATE

    def write_src_file(self, fname, content):
        with open(os.path.join(self.srcdir, fname), "w") as f:
            f.write(content)

    def setUp(self):
        if os.path.exists(self.workdir):
            shutil.rmtree(self.workdir)
        os.mkdir(self.workdir)
        os.mkdir(self.srcdir)
        os.mkdir(self.bindir)
        self.write_src_file("CMakeLists.txt", self.gen_cmakelists())
        self.write_src_file("main.cpp", self.gen_code())
        os.chdir(self.bindir)

    def tearDown(self):
        pass
        #if os.path.exists(self.workdir):
        #    shutil.rmtree(self.workdir)

    def runTest(self):
        cmd = [
            "cmake",
            "-GNinja",
            "-DOpenCV_DIR=%s" % self.opencv_cmake_path,
            "-DCMAKE_TOOLCHAIN_FILE=%s" % self.getCMakeToolchain(),
            self.srcdir
        ] + [ "-D{}={}".format(key, value) for key, value in self.cmake_vars.items() ]
        log.info("Executing: %s" % cmd)
        retcode = subprocess.call(cmd)
        self.assertEqual(retcode, 0, "cmake failed")

        cmd = ["ninja", "-v"]
        log.info("Executing: %s" % cmd)
        retcode = subprocess.call(cmd)
        self.assertEqual(retcode, 0, "make failed")

def suite(workdir, opencv_cmake_path):
    abis = {
        "armeabi-v7a": { "ANDROID_ABI": "armeabi-v7a", "ANDROID_TOOLCHAIN": "clang", "ANDROID_STL": "c++_shared", 'ANDROID_NATIVE_API_LEVEL': "21" },
        "arm64-v8a": { "ANDROID_ABI": "arm64-v8a", "ANDROID_TOOLCHAIN": "clang", "ANDROID_STL": "c++_shared", 'ANDROID_NATIVE_API_LEVEL': "21" },
        "x86": { "ANDROID_ABI": "x86", "ANDROID_TOOLCHAIN": "clang", "ANDROID_STL": "c++_shared", 'ANDROID_NATIVE_API_LEVEL': "21" },
        "x86_64": { "ANDROID_ABI": "x86_64", "ANDROID_TOOLCHAIN": "clang", "ANDROID_STL": "c++_shared", 'ANDROID_NATIVE_API_LEVEL': "21" },
    }

    suite = unittest.TestSuite()
    for libset in ["", "opencv_java"]:
        for abi, cmake_vars in abis.items():
            suite.addTest(TestCmakeBuild(libset, abi, cmake_vars, opencv_cmake_path,
                    os.path.join(workdir, "{}-{}".format(abi, "static" if libset == "" else "shared"))))
    return suite


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Test OpenCV for Android SDK with cmake')
    parser.add_argument('--sdk_path', help="Path to Android SDK to use for build")
    parser.add_argument('--ndk_path', help="Path to Android NDK to use for build")
    parser.add_argument("--workdir", default="testspace", help="Working directory (and output)")
    parser.add_argument("opencv_cmake_path", help="Path to folder with OpenCVConfig.cmake and android.toolchain.cmake (usually <SDK>/sdk/native/jni/")

    args = parser.parse_args()

    if args.sdk_path is not None:
        os.environ["ANDROID_SDK"] = os.path.abspath(args.sdk_path)
    if args.ndk_path is not None:
        os.environ["ANDROID_NDK"] = os.path.abspath(args.ndk_path)

    if not 'ANDROID_HOME' in os.environ and 'ANDROID_SDK' in os.environ:
        os.environ['ANDROID_HOME'] = os.environ["ANDROID_SDK"]

    print("Using SDK: %s" % os.environ["ANDROID_SDK"])
    print("Using NDK: %s" % os.environ["ANDROID_NDK"])

    workdir = os.path.abspath(args.workdir)
    if not os.path.exists(workdir):
        os.mkdir(workdir)
    res = unittest.TextTestRunner(verbosity=3).run(suite(workdir, os.path.abspath(args.opencv_cmake_path)))
    if not res.wasSuccessful():
        sys.exit(res)
```

## High-Level Overview

This is a Python file that may contain scripts, bindings, or utilities.

**Key Characteristics:**
- May provide Python bindings to C++ code
- Could be a utility script for build/test automation
- Might implement examples or tutorials
- Uses Python idioms and standard library


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **TestCmakeBuild**: A class/struct defined in this file

### Functions and Methods

- **tearDown()**: A function/method defined in this file
- **gen_cmakelists()**: A function/method defined in this file
- **gen_code()**: A function/method defined in this file
- **runTest()**: A function/method defined in this file
- **suite()**: A function/method defined in this file
- **shortDescription()**: A function/method defined in this file
- **write_src_file()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **getCMakeToolchain()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`
- `opencv2/imgproc.hpp`

**Python Imports:**
- `os`
- `logging`
- `unittest`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

