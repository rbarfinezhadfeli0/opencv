# Documentation for `docs/platforms/ios/build_visionos_framework.py_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/ios/build_visionos_framework.py_docs.md`
- **File Name**: `build_visionos_framework.py_docs.md`
- **File Size**: 10,379 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/ios/build_visionos_framework.py_docs.md](../../../docs/platforms/ios/build_visionos_framework.py_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/ios` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/ios/build_visionos_framework.py`

## File Metadata

- **Full Path**: `platforms/ios/build_visionos_framework.py`
- **File Name**: `build_visionos_framework.py`
- **File Size**: 6,803 bytes
- **File Type**: .py
- **Link to Source**: [platforms/ios/build_visionos_framework.py](../../platforms/ios/build_visionos_framework.py)

## Purpose and Role

This file is located in the `platforms/ios` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
"""
The script builds OpenCV.framework for visionOS.
"""

from __future__ import print_function
import os, os.path, sys, argparse, traceback, multiprocessing

# import common code
# sys.path.insert(0, os.path.abspath(os.path.abspath(os.path.dirname(__file__))+'/../ios'))
from build_framework import Builder
sys.path.insert(0, os.path.abspath(os.path.abspath(os.path.dirname(__file__))+'/../apple'))
from cv_build_utils import print_error, get_cmake_version

XROS_DEPLOYMENT_TARGET='1.0'  # default, can be changed via command line options or environment variable

class visionOSBuilder(Builder):

    def checkCMakeVersion(self):
        assert get_cmake_version() >= (3, 17), "CMake 3.17 or later is required. Current version is {}".format(get_cmake_version())

    def getObjcTarget(self, target):
        return 'visionos'

    def getToolchain(self, arch, target):
        toolchain = os.path.join(self.opencv, "platforms", "ios", "cmake", "Toolchains", "Toolchain-%s_Xcode.cmake" % target)
        return toolchain

    def getCMakeArgs(self, arch, target):
        args = Builder.getCMakeArgs(self, arch, target)
        args = args + [
            '-DVISIONOS_ARCH=%s' % arch
        ]
        return args

    def getBuildCommand(self, arch, target):
        buildcmd = [
            "xcodebuild",
            "XROS_DEPLOYMENT_TARGET=" + os.environ['XROS_DEPLOYMENT_TARGET'],
            "ARCHS=%s" % arch,
            "-sdk", target.lower(),
            "-configuration", "Debug" if self.debug else "Release",
            "-parallelizeTargets",
            "-jobs", str(multiprocessing.cpu_count())
        ]

        return buildcmd

    def getInfoPlist(self, builddirs):
        return os.path.join(builddirs[0], "visionos", "Info.plist")


if __name__ == "__main__":
    folder = os.path.abspath(os.path.join(os.path.dirname(sys.argv[0]), "../.."))
    parser = argparse.ArgumentParser(description='The script builds OpenCV.framework for visionOS.')
    # TODO: When we can make breaking changes, we should make the out argument explicit and required like in build_xcframework.py.
    parser.add_argument('out', metavar='OUTDIR', help='folder to put built framework')
    parser.add_argument('--opencv', metavar='DIR', default=folder, help='folder with opencv repository (default is "../.." relative to script location)')
    parser.add_argument('--contrib', metavar='DIR', default=None, help='folder with opencv_contrib repository (default is "None" - build only main framework)')
    parser.add_argument('--without', metavar='MODULE', default=[], action='append', help='OpenCV modules to exclude from the framework. To exclude multiple, specify this flag again, e.g. "--without video --without objc"')
    parser.add_argument('--disable', metavar='FEATURE', default=[], action='append', help='OpenCV features to disable (add WITH_*=OFF). To disable multiple, specify this flag again, e.g. "--disable tbb --disable openmp"')
    parser.add_argument('--dynamic', default=False, action='store_true', help='build dynamic framework (default is "False" - builds static framework)')
    parser.add_argument('--enable_nonfree', default=False, dest='enablenonfree', action='store_true', help='enable non-free modules (disabled by default)')
    parser.add_argument('--visionos_deployment_target', default=os.environ.get('XROS_DEPLOYMENT_TARGET', XROS_DEPLOYMENT_TARGET), help='specify XROS_DEPLOYMENT_TARGET')
    parser.add_argument('--visionos_archs', default=None, help='select visionOS target ARCHS. Default is none')
    parser.add_argument('--visionsimulator_archs', default=None, help='select visionSimulator target ARCHS. Default is none')
    parser.add_argument('--debug', action='store_true', help='Build "Debug" binaries (CMAKE_BUILD_TYPE=Debug)')
    parser.add_argument('--debug_info', action='store_true', help='Build with debug information (useful for Release mode: BUILD_WITH_DEBUG_INFO=ON)')
    parser.add_argument('--framework_name', default='opencv2', dest='framework_name', help='Name of OpenCV framework (default: opencv2, will change to OpenCV in future version)')
    parser.add_argument('--legacy_build', default=False, dest='legacy_build', action='store_true', help='Build legacy framework (default: False, equivalent to "--framework_name=opencv2 --without=objc")')
    parser.add_argument('--run_tests', default=False, dest='run_tests', action='store_true', help='Run tests')
    parser.add_argument('--build_docs', default=False, dest='build_docs', action='store_true', help='Build docs')
    parser.add_argument('--disable-swift', default=False, dest='swiftdisabled', action='store_true', help='Disable building of Swift extensions')

    args, unknown_args = parser.parse_known_args()
    if unknown_args:
        print("The following args are not recognized and will not be used: %s" % unknown_args)

    os.environ['XROS_DEPLOYMENT_TARGET'] = args.visionos_deployment_target
    print('Using XROS_DEPLOYMENT_TARGET=' + os.environ['XROS_DEPLOYMENT_TARGET'])

    visionos_archs = None
    if args.visionos_archs:
        visionos_archs = args.visionos_archs.split(',')
    print('Using visionOS ARCHS=' + str(visionos_archs))

    visionsimulator_archs = None
    if args.visionsimulator_archs:
        visionsimulator_archs = args.visionsimulator_archs.split(',')
    print('Using visionOS ARCHS=' + str(visionsimulator_archs))

    # Prevent the build from happening if the same architecture is specified for multiple platforms.
    # When `lipo` is run to stitch the frameworks together into a fat framework, it'll fail, so it's
    # better to stop here while we're ahead.
    if visionos_archs and visionsimulator_archs:
        duplicate_archs = set(visionos_archs).intersection(visionsimulator_archs)
        if duplicate_archs:
            print_error("Cannot have the same architecture for multiple platforms in a fat framework! Consider using build_xcframework.py in the apple platform folder instead. Duplicate archs are %s" % duplicate_archs)
            exit(1)

    if args.legacy_build:
        args.framework_name = "opencv2"
        if not "objc" in args.without:
            args.without.append("objc")

    targets = []
    if not visionos_archs and not visionsimulator_archs:
        print_error("--visionos_archs and --visionsimulator_archs are undefined; nothing will be built.")
        sys.exit(1)
    if visionos_archs:
        targets.append((visionos_archs, "XROS"))
    if visionsimulator_archs:
        targets.append((visionsimulator_archs, "XRSimulator")),

    b = visionOSBuilder(args.opencv, args.contrib, args.dynamic, True, args.without, args.disable, args.enablenonfree, targets, args.debug, args.debug_info, args.framework_name, args.run_tests, args.build_docs, args.swiftdisabled)
    b.build(args.out)
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

- **visionOSBuilder**: A class/struct defined in this file

### Functions and Methods

- **import()**: A function/method defined in this file
- **getBuildCommand()**: A function/method defined in this file
- **getObjcTarget()**: A function/method defined in this file
- **getInfoPlist()**: A function/method defined in this file
- **getCMakeArgs()**: A function/method defined in this file
- **getToolchain()**: A function/method defined in this file
- **checkCMakeVersion()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `the`
- `Builder`
- `happening`
- `os`
- `cv_build_utils`
- `print_error`
- `print_function`
- `common`
- `build_framework`
- `__future__`


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

