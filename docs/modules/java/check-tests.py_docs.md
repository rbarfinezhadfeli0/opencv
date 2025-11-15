# Documentation for `modules/java/check-tests.py`

## File Metadata

- **Full Path**: `modules/java/check-tests.py`
- **File Name**: `check-tests.py`
- **File Size**: 5,747 bytes
- **File Type**: .py
- **Link to Source**: [modules/java/check-tests.py](../../modules/java/check-tests.py)

## Purpose and Role

This file is located in the `modules/java` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

from __future__ import print_function
import sys, os, re

classes_ignore_list = (
    'OpenCV(Test)?Case',
    'OpenCV(Test)?Runner',
    'CvException',
)

funcs_ignore_list = (
    '\w+--HashCode',
    'Mat--MatLong',
    '\w+--Equals',
    'Core--MinMaxLocResult',
)

class JavaParser:
    def __init__(self):
        self.clear()

    def clear(self):
        self.mdict = {}
        self.tdict = {}
        self.mwhere = {}
        self.twhere = {}
        self.empty_stubs_cnt = 0
        self.r1 = re.compile("\s*public\s+(?:static\s+)?(\w+)\(([^)]*)\)") # c-tor
        self.r2 = re.compile("\s*(?:(?:public|static|final)\s+){1,3}\S+\s+(\w+)\(([^)]*)\)")
        self.r3 = re.compile('\s*fail\("Not yet implemented"\);') # empty test stub


    def dict2set(self, d):
        s = set()
        for f in d.keys():
            if len(d[f]) == 1:
                s.add(f)
            else:
                s |= set(d[f])
        return s


    def get_tests_count(self):
        return len(self.tdict)

    def get_empty_stubs_count(self):
        return self.empty_stubs_cnt

    def get_funcs_count(self):
        return len(self.dict2set(self.mdict)), len(self.mdict)

    def get_not_tested(self):
        mset = self.dict2set(self.mdict)
        tset = self.dict2set(self.tdict)
        nottested = mset - tset
        out = set()

        for name in nottested:
            out.add(name + "   " + self.mwhere[name])

        return out


    def parse(self, path):
        if ".svn" in path:
            return
        if os.path.isfile(path):
            if path.endswith("FeatureDetector.java"):
                for prefix1 in ("", "Grid", "Pyramid", "Dynamic"):
                    for prefix2 in ("FAST", "STAR", "MSER", "ORB", "SIFT", "SURF", "GFTT", "HARRIS", "SIMPLEBLOB", "DENSE", "AKAZE", "KAZE", "BRISK", "AGAST"):
                        parser.parse_file(path,prefix1+prefix2)
            elif path.endswith("DescriptorExtractor.java"):
                for prefix1 in ("", "Opponent"):
                    for prefix2 in ("BRIEF", "ORB", "SIFT", "SURF", "AKAZE", "KAZE", "BEBLID", "DAISY", "FREAK", "LUCID", "LATCH"):
                        parser.parse_file(path,prefix1+prefix2)
            elif path.endswith("GenericDescriptorMatcher.java"):
                for prefix in ("OneWay", "Fern"):
                    parser.parse_file(path,prefix)
            elif path.endswith("DescriptorMatcher.java"):
                for prefix in ("BruteForce", "BruteForceHamming", "BruteForceHammingLUT", "BruteForceL1", "FlannBased", "BruteForceSL2"):
                    parser.parse_file(path,prefix)
            else:
                parser.parse_file(path)
        elif os.path.isdir(path):
            for x in os.listdir(path):
                self.parse(path + "/" + x)
        return


    def parse_file(self, fname, prefix = ""):
        istest = fname.endswith("Test.java")
        clsname = os.path.basename(fname).replace("Test", "").replace(".java", "")
        clsname = prefix + clsname[0].upper() + clsname[1:]
        for cls in classes_ignore_list:
            if re.match(cls, clsname):
                return
        f = open(fname, "rt")
        linenum = 0
        for line in f:
            linenum += 1
            m1 = self.r1.match(line)
            m2 = self.r2.match(line)
            m3 = self.r3.match(line)
            func = ''
            args_str = ''
            if m1:
                func = m1.group(1)
                args_str = m1.group(2)
            elif m2:
                if "public" not in line:
                    continue
                func = m2.group(1)
                args_str = m2.group(2)
            elif m3:
                self.empty_stubs_cnt += 1
                continue
            else:
                #if "public" in line:
                    #print "UNRECOGNIZED: " + line
                continue
            d = (self.mdict, self.tdict)[istest]
            w = (self.mwhere, self.twhere)[istest]
            func = re.sub(r"^test", "", func)
            func = clsname + "--" + func[0].upper() + func[1:]
            args_str = args_str.replace("[]", "Array").replace("...", "Array ")
            args_str = re.sub(r"List<(\w+)>", "ListOf\g<1>", args_str)
            args_str = re.sub(r"List<(\w+)>", "ListOf\g<1>", args_str)
            args = [a.split()[0] for a in args_str.split(",") if a]
            func_ex = func + "".join([a[0].upper() + a[1:] for a in args])
            func_loc = fname + " (line: " + str(linenum)  + ")"
            skip = False
            for fi in funcs_ignore_list:
                if re.match(fi, func_ex):
                    skip = True
                    break
            if skip:
                continue
            if func in d:
                d[func].append(func_ex)
            else:
                d[func] = [func_ex]
            w[func_ex] = func_loc
            w[func] = func_loc

        f.close()
        return


if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("Usage:\n", \
            os.path.basename(sys.argv[0]), \
            "<Classes/Tests dir1/file1> [<Classes/Tests dir2/file2> ...]\n", "Not tested methods are loggedto stdout.")
        exit(0)
    parser = JavaParser()
    for x in sys.argv[1:]:
        parser.parse(x)
    funcs = parser.get_not_tested()
    if funcs:
        print ('{} {}'.format("NOT TESTED methods:\n\t", "\n\t".join(sorted(funcs))))
    print ("Total methods found: %i (%i)" % parser.get_funcs_count())
    print ('{} {}'.format("Not tested methods found:", len(funcs)))
    print ('{} {}'.format("Total tests found:", parser.get_tests_count()))
    print ('{} {}'.format("Empty test stubs found:", parser.get_empty_stubs_count()))
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

- **JavaParser**: A class/struct defined in this file

### Functions and Methods

- **get_tests_count()**: A function/method defined in this file
- **parse()**: A function/method defined in this file
- **in()**: A function/method defined in this file
- **get_not_tested()**: A function/method defined in this file
- **clear()**: A function/method defined in this file
- **dict2set()**: A function/method defined in this file
- **get_funcs_count()**: A function/method defined in this file
- **parse_file()**: A function/method defined in this file
- **get_empty_stubs_count()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `print_function`
- `__future__`
- `sys`


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

