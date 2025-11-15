# Documentation for `modules/ts/misc/run_android.py`

## File Metadata

- **Full Path**: `modules/ts/misc/run_android.py`
- **File Name**: `run_android.py`
- **File Size**: 6,884 bytes
- **File Type**: .py
- **Link to Source**: [modules/ts/misc/run_android.py](../../../modules/ts/misc/run_android.py)

## Purpose and Role

This file is located in the `modules/ts/misc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
""" Utility package for run.py
"""

import os
import re
import getpass
from run_utils import Err, log, execute, isColorEnabled, hostos
from run_suite import TestSuite


def exe(program):
    return program + ".exe" if hostos == 'nt' else program


class ApkInfo:
    def __init__(self):
        self.pkg_name = None
        self.pkg_target = None
        self.pkg_runner = None

    def forcePackage(self, package):
        if package:
            if package.startswith("."):
                self.pkg_target += package
            else:
                self.pkg_target = package


class Tool:
    def __init__(self):
        self.cmd = []

    def run(self, args=[], silent=False):
        cmd = self.cmd[:]
        cmd.extend(args)
        return execute(self.cmd + args, silent)


class Adb(Tool):
    def __init__(self, sdk_dir):
        Tool.__init__(self)
        exe_path = os.path.join(sdk_dir, exe("platform-tools/adb"))
        if not os.path.isfile(exe_path) or not os.access(exe_path, os.X_OK):
            exe_path = None
        # fix adb tool location
        if not exe_path:
            exe_path = "adb"
        self.cmd = [exe_path]

    def init(self, serial):
        # remember current device serial. Needed if another device is connected while this script runs
        if not serial:
            serial = self.detectSerial()
        if serial:
            self.cmd.extend(["-s", serial])

    def detectSerial(self):
        adb_res = self.run(["devices"], silent=True)
        # assume here that device name may consists of any characters except newline
        connected_devices = re.findall(r"^[^\n]+[ \t]+device\r?$", adb_res, re.MULTILINE)
        if not connected_devices:
            raise Err("Can not find Android device")
        elif len(connected_devices) != 1:
            raise Err("Too many (%s) devices are connected. Please specify single device using --serial option:\n\n%s", len(connected_devices), adb_res)
        else:
            return connected_devices[0].split("\t")[0]

    def getOSIdentifier(self):
        return "Android" + self.run(["shell", "getprop ro.build.version.release"], silent=True).strip()


class Aapt(Tool):
    def __init__(self, sdk_dir):
        Tool.__init__(self)
        aapt_fn = exe("aapt")
        aapt = None
        for r, ds, fs in os.walk(os.path.join(sdk_dir, 'build-tools')):
            if aapt_fn in fs:
                aapt = os.path.join(r, aapt_fn)
                break
        if not aapt:
            raise Err("Can not find aapt tool: %s", aapt_fn)
        self.cmd = [aapt]

    def dump(self, exe):
        res = ApkInfo()
        output = self.run(["dump", "xmltree", exe, "AndroidManifest.xml"], silent=True)
        if not output:
            raise Err("Can not dump manifest from %s", exe)
        tags = re.split(r"[ ]+E: ", output)
        # get package name
        manifest_tag = [t for t in tags if t.startswith("manifest ")]
        if not manifest_tag:
            raise Err("Can not read package name from: %s", exe)
        res.pkg_name = re.search(r"^[ ]+A: package=\"(?P<pkg>.*?)\" \(Raw: \"(?P=pkg)\"\)\r?$", manifest_tag[0], flags=re.MULTILINE).group("pkg")
        # get test instrumentation info
        instrumentation_tag = [t for t in tags if t.startswith("instrumentation ")]
        if not instrumentation_tag:
            raise Err("Can not find instrumentation details in: %s", exe)
        res.pkg_runner = re.search(r"^[ ]+A: android:name\(0x[0-9a-f]{8}\)=\"(?P<runner>.*?)\" \(Raw: \"(?P=runner)\"\)\r?$", instrumentation_tag[0], flags=re.MULTILINE).group("runner")
        res.pkg_target = re.search(r"^[ ]+A: android:targetPackage\(0x[0-9a-f]{8}\)=\"(?P<pkg>.*?)\" \(Raw: \"(?P=pkg)\"\)\r?$", instrumentation_tag[0], flags=re.MULTILINE).group("pkg")
        if not res.pkg_name or not res.pkg_runner or not res.pkg_target:
            raise Err("Can not find instrumentation details in: %s", exe)
        return res


class AndroidTestSuite(TestSuite):
    def __init__(self, options, cache, id, android_env={}):
        TestSuite.__init__(self, options, cache, id)
        sdk_dir = options.android_sdk or os.environ.get("ANDROID_SDK", False) or os.path.dirname(os.path.dirname(self.cache.android_executable))
        log.debug("Detecting Android tools in directory: %s", sdk_dir)
        self.adb = Adb(sdk_dir)
        self.aapt = Aapt(sdk_dir)
        self.env = android_env

    def isTest(self, fullpath):
        if os.path.isfile(fullpath):
            if fullpath.endswith(".apk") or os.access(fullpath, os.X_OK):
                return True
        return False

    def getOS(self):
        return self.adb.getOSIdentifier()

    def checkPrerequisites(self):
        self.adb.init(self.options.serial)

    def runTest(self, module, path, logfile, workingDir, args=[]):
        args = args[:]
        exe = os.path.abspath(path)

        if exe.endswith(".apk"):
            info = self.aapt.dump(exe)
            if not info:
                raise Err("Can not read info from test package: %s", exe)
            info.forcePackage(self.options.package)
            self.adb.run(["uninstall", info.pkg_name])

            output = self.adb.run(["install", exe], silent=True)
            if not (output and "Success" in output):
                raise Err("Can not install package: %s", exe)

            params = ["-e package %s" % info.pkg_target]
            ret = self.adb.run(["shell", "am instrument -w %s %s/%s" % (" ".join(params), info.pkg_name, info.pkg_runner)])
            return None, ret
        else:
            device_dir = getpass.getuser().replace(" ", "") + "_" + self.options.mode + "/"
            if isColorEnabled(args):
                args.append("--gtest_color=yes")
            tempdir = "/data/local/tmp/"
            android_dir = tempdir + device_dir
            exename = os.path.basename(exe)
            android_exe = android_dir + exename
            self.adb.run(["push", exe, android_exe])
            self.adb.run(["shell", "chmod 777 " + android_exe])
            env_pieces = ["export %s=%s" % (a, b) for a, b in self.env.items()]
            pieces = ["cd %s" % android_dir, "./%s %s" % (exename, " ".join(args))]
            log.warning("Run: %s" % " && ".join(pieces))
            ret = self.adb.run(["shell", " && ".join(env_pieces + pieces)])
            # try get log
            hostlogpath = os.path.join(workingDir, logfile)
            self.adb.run(["pull", android_dir + logfile, hostlogpath])
            # cleanup
            self.adb.run(["shell", "rm " + android_dir + logfile])
            self.adb.run(["shell", "rm " + tempdir + "__opencv_temp.*"], silent=True)
            if os.path.isfile(hostlogpath):
                return hostlogpath, ret
            return None, ret


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

- **Tool**: A class/struct defined in this file
- **Adb**: A class/struct defined in this file
- **Aapt**: A class/struct defined in this file
- **ApkInfo**: A class/struct defined in this file
- **AndroidTestSuite**: A class/struct defined in this file

### Functions and Methods

- **getOS()**: A function/method defined in this file
- **in()**: A function/method defined in this file
- **forcePackage()**: A function/method defined in this file
- **isTest()**: A function/method defined in this file
- **checkPrerequisites()**: A function/method defined in this file
- **runTest()**: A function/method defined in this file
- **getOSIdentifier()**: A function/method defined in this file
- **dump()**: A function/method defined in this file
- **exe()**: A function/method defined in this file
- **run()**: A function/method defined in this file
- **init()**: A function/method defined in this file
- **detectSerial()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `run_utils`
- `os`
- `re`
- `test`
- `Err`
- `run_suite`
- `TestSuite`
- `getpass`


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

