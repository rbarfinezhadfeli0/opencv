# Documentation for `modules/ts/misc/run_utils.py`

## File Metadata

- **Full Path**: `modules/ts/misc/run_utils.py`
- **File Name**: `run_utils.py`
- **File Size**: 7,391 bytes
- **File Type**: .py
- **Link to Source**: [modules/ts/misc/run_utils.py](../../../modules/ts/misc/run_utils.py)

## Purpose and Role

This file is located in the `modules/ts/misc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
""" Utility package for run.py
"""

import sys
import os
import platform
import re
import tempfile
import glob
import logging
import shutil
from subprocess import check_call, check_output, CalledProcessError, STDOUT


def initLogger():
    logger = logging.getLogger("run.py")
    logger.setLevel(logging.DEBUG)
    ch = logging.StreamHandler(sys.stderr)
    ch.setFormatter(logging.Formatter("%(message)s"))
    logger.addHandler(ch)
    return logger


log = initLogger()
hostos = os.name  # 'nt', 'posix'


class Err(Exception):
    def __init__(self, msg, *args):
        self.msg = msg % args


def execute(cmd, silent=False, cwd=".", env=None):
    try:
        log.debug("Run: %s", cmd)
        if env is not None:
            for k in env:
                log.debug("    Environ: %s=%s", k, env[k])
            new_env = os.environ.copy()
            new_env.update(env)
            env = new_env

        if sys.platform == 'darwin':  # https://github.com/opencv/opencv/issues/14351
            if env is None:
                env = os.environ.copy()
            if 'DYLD_LIBRARY_PATH' in env:
                env['OPENCV_SAVED_DYLD_LIBRARY_PATH'] = env['DYLD_LIBRARY_PATH']

        if silent:
            return check_output(cmd, stderr=STDOUT, cwd=cwd, env=env).decode("latin-1")
        else:
            return check_call(cmd, cwd=cwd, env=env)
    except CalledProcessError as e:
        if silent:
            log.debug("Process returned: %d", e.returncode)
            return e.output.decode("latin-1")
        else:
            log.error("Process returned: %d", e.returncode)
            return e.returncode


def isColorEnabled(args):
    usercolor = [a for a in args if a.startswith("--gtest_color=")]
    return len(usercolor) == 0 and sys.stdout.isatty() and hostos != "nt"


def getPlatformVersion():
    mv = platform.mac_ver()
    if mv[0]:
        return "Darwin" + mv[0]
    else:
        wv = platform.win32_ver()
        if wv[0]:
            return "Windows" + wv[0]
        else:
            lv = platform.linux_distribution()
            if lv[0]:
                return lv[0] + lv[1]
    return None


parse_patterns = (
    {'name': "cmake_home",               'default': None,       'pattern': re.compile(r"^CMAKE_HOME_DIRECTORY:\w+=(.+)$")},
    {'name': "opencv_home",              'default': None,       'pattern': re.compile(r"^OpenCV_SOURCE_DIR:\w+=(.+)$")},
    {'name': "opencv_build",             'default': None,       'pattern': re.compile(r"^OpenCV_BINARY_DIR:\w+=(.+)$")},
    {'name': "tests_dir",                'default': None,       'pattern': re.compile(r"^EXECUTABLE_OUTPUT_PATH:\w+=(.+)$")},
    {'name': "build_type",               'default': "Release",  'pattern': re.compile(r"^CMAKE_BUILD_TYPE:\w+=(.*)$")},
    {'name': "android_abi",              'default': None,       'pattern': re.compile(r"^ANDROID_ABI:\w+=(.*)$")},
    {'name': "android_executable",       'default': None,       'pattern': re.compile(r"^ANDROID_EXECUTABLE:\w+=(.*android.*)$")},
    {'name': "ant_executable",           'default': None,       'pattern': re.compile(r"^ANT_EXECUTABLE:\w+=(.*ant.*)$")},
    {'name': "java_test_dir",            'default': None,       'pattern': re.compile(r"^OPENCV_JAVA_TEST_DIR:\w+=(.*)$")},
    {'name': "is_x64",                   'default': "OFF",      'pattern': re.compile(r"^CUDA_64_BIT_DEVICE_CODE:\w+=(ON)$")},
    {'name': "cmake_generator",          'default': None,       'pattern': re.compile(r"^CMAKE_GENERATOR:\w+=(.+)$")},
    {'name': "python2",                  'default': None,       'pattern': re.compile(r"^BUILD_opencv_python2:\w+=(.*)$")},
    {'name': "python3",                  'default': None,       'pattern': re.compile(r"^BUILD_opencv_python3:\w+=(.*)$")},
)


class CMakeCache:
    def __init__(self, cfg=None):
        self.setDefaultAttrs()
        self.main_modules = []
        if cfg:
            self.build_type = cfg

    def setDummy(self, path):
        self.tests_dir = os.path.normpath(path)

    def read(self, path, fname):
        rx = re.compile(r'^OPENCV_MODULE_opencv_(\w+)_LOCATION:INTERNAL=(.*)$')
        module_paths = {}  # name -> path
        with open(fname, "rt") as cachefile:
            for l in cachefile.readlines():
                ll = l.strip()
                if not ll or ll.startswith("#"):
                    continue
                for p in parse_patterns:
                    match = p["pattern"].match(ll)
                    if match:
                        value = match.groups()[0]
                        if value and not value.endswith("-NOTFOUND"):
                            setattr(self, p["name"], value)
                            # log.debug("cache value: %s = %s", p["name"], value)

                match = rx.search(ll)
                if match:
                    module_paths[match.group(1)] = match.group(2)

        if not self.tests_dir:
            self.tests_dir = path
        else:
            rel = os.path.relpath(self.tests_dir, self.opencv_build)
            self.tests_dir = os.path.join(path, rel)
        self.tests_dir = os.path.normpath(self.tests_dir)

        # fix VS test binary path (add Debug or Release)
        if "Visual Studio" in self.cmake_generator:
            self.tests_dir = os.path.join(self.tests_dir, self.build_type)

        for module, path in module_paths.items():
            rel = os.path.relpath(path, self.opencv_home)
            if ".." not in rel:
                self.main_modules.append(module)

    def setDefaultAttrs(self):
        for p in parse_patterns:
            setattr(self, p["name"], p["default"])

    def gatherTests(self, mask, isGood=None):
        if self.tests_dir and os.path.isdir(self.tests_dir):
            d = os.path.abspath(self.tests_dir)
            files = glob.glob(os.path.join(d, mask))
            if not self.getOS() == "android" and self.withJava():
                files.append("java")
            if self.withPython2():
                files.append("python2")
            if self.withPython3():
                files.append("python3")
            return [f for f in files if isGood(f)]
        return []

    def isMainModule(self, name):
        return name in self.main_modules + ['python2', 'python3']

    def withJava(self):
        return self.ant_executable and self.java_test_dir and os.path.exists(self.java_test_dir)

    def withPython2(self):
        return self.python2 == 'ON'

    def withPython3(self):
        return self.python3 == 'ON'

    def getOS(self):
        if self.android_executable:
            return "android"
        else:
            return hostos


class TempEnvDir:
    def __init__(self, envname, prefix):
        self.envname = envname
        self.prefix = prefix
        self.saved_name = None
        self.new_name = None

    def init(self):
        self.saved_name = os.environ.get(self.envname)
        self.new_name = tempfile.mkdtemp(prefix=self.prefix, dir=self.saved_name or None)
        os.environ[self.envname] = self.new_name

    def clean(self):
        if self.saved_name:
            os.environ[self.envname] = self.saved_name
        else:
            del os.environ[self.envname]
        try:
            shutil.rmtree(self.new_name)
        except:
            pass


if __name__ == "__main__":
    log.error("This is utility file, please execute run.py script")
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

- **CMakeCache**: A class/struct defined in this file
- **TempEnvDir**: A class/struct defined in this file
- **Err**: A class/struct defined in this file

### Functions and Methods

- **initLogger()**: A function/method defined in this file
- **getOS()**: A function/method defined in this file
- **setDummy()**: A function/method defined in this file
- **isColorEnabled()**: A function/method defined in this file
- **gatherTests()**: A function/method defined in this file
- **withPython2()**: A function/method defined in this file
- **withPython3()**: A function/method defined in this file
- **withJava()**: A function/method defined in this file
- **setDefaultAttrs()**: A function/method defined in this file
- **isMainModule()**: A function/method defined in this file
- **clean()**: A function/method defined in this file
- **getPlatformVersion()**: A function/method defined in this file
- **execute()**: A function/method defined in this file
- **init()**: A function/method defined in this file
- **read()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tempfile`
- `glob`
- `sys`
- `os`
- `re`
- `check_call`
- `platform`
- `logging`
- `shutil`
- `subprocess`


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

