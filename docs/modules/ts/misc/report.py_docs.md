# Documentation for `modules/ts/misc/report.py`

## File Metadata

- **Full Path**: `modules/ts/misc/report.py`
- **File Name**: `report.py`
- **File Size**: 5,853 bytes
- **File Type**: .py
- **Link to Source**: [modules/ts/misc/report.py](../../../modules/ts/misc/report.py)

## Purpose and Role

This file is located in the `modules/ts/misc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
""" Print performance test run statistics.

Performance data is stored in the GTest log file created by performance tests. Default name is
`test_details.xml`. It can be changed with the `--gtest_output=xml:<location>/<filename>.xml` test
option. See https://github.com/opencv/opencv/wiki/HowToUsePerfTests for more details.

This script produces configurable performance report tables in text and HTML formats. It allows to
filter test cases by name and parameter string and select specific performance metrics columns. One
or multiple test results can be used for input.

### Example

./report.py  -c min,mean,median -f '(LUT|Match).*640' opencv_perf_core.xml  opencv_perf_features2d.xml

opencv_perf_features2d.xml, opencv_perf_core.xml

                       Name of Test                            Min        Mean      Median
KnnMatch::OCL_BruteForceMatcherFixture::(640x480, 32FC1)    1365.04 ms 1368.18 ms 1368.52 ms
LUT::OCL_LUTFixture::(640x480, 32FC1)                        2.57 ms    2.62 ms    2.64 ms
LUT::OCL_LUTFixture::(640x480, 32FC4)                        21.15 ms   21.25 ms   21.24 ms
LUT::OCL_LUTFixture::(640x480, 8UC1)                         2.22 ms    2.28 ms    2.29 ms
LUT::OCL_LUTFixture::(640x480, 8UC4)                         19.12 ms   19.24 ms   19.19 ms
LUT::SizePrm::640x480                                        2.22 ms    2.27 ms    2.29 ms
Match::OCL_BruteForceMatcherFixture::(640x480, 32FC1)       1364.15 ms 1367.73 ms 1365.45 ms
RadiusMatch::OCL_BruteForceMatcherFixture::(640x480, 32FC1) 1372.68 ms 1375.52 ms 1375.42 ms

### Options

-o FMT, --output=FMT        - output results in text format (can be 'txt', 'html' or 'auto' - default)
-u UNITS, --units=UNITS     - units for output values (s, ms (default), us, ns or ticks)
-c COLS, --columns=COLS     - comma-separated list of columns to show
-f REGEX, --filter=REGEX    - regex to filter tests
--show-all                  - also include empty and "notrun" lines
"""

from __future__ import print_function
import testlog_parser, sys, os, xml, re, glob
from table_formatter import *
from optparse import OptionParser

if __name__ == "__main__":
    parser = OptionParser()
    parser.add_option("-o", "--output", dest="format", help="output results in text format (can be 'txt', 'html' or 'auto' - default)", metavar="FMT", default="auto")
    parser.add_option("-u", "--units", dest="units", help="units for output values (s, ms (default), us, ns or ticks)", metavar="UNITS", default="ms")
    parser.add_option("-c", "--columns", dest="columns", help="comma-separated list of columns to show", metavar="COLS", default="")
    parser.add_option("-f", "--filter", dest="filter", help="regex to filter tests", metavar="REGEX", default=None)
    parser.add_option("", "--show-all", action="store_true", dest="showall", default=False, help="also include empty and \"notrun\" lines")
    (options, args) = parser.parse_args()

    if len(args) < 1:
        print("Usage:\n", os.path.basename(sys.argv[0]), "<log_name1>.xml", file=sys.stderr)
        exit(0)

    options.generateHtml = detectHtmlOutputType(options.format)

    # expand wildcards and filter duplicates
    files = []
    files1 = []
    for arg in args:
        if ("*" in arg) or ("?" in arg):
            files1.extend([os.path.abspath(f) for f in glob.glob(arg)])
        else:
            files.append(os.path.abspath(arg))
    seen = set()
    files = [ x for x in files if x not in seen and not seen.add(x)]
    files.extend((set(files1) - set(files)))
    args = files

    # load test data
    tests = []
    files = []
    for arg in set(args):
        try:
            cases = testlog_parser.parseLogFile(arg)
            if cases:
                files.append(os.path.basename(arg))
                tests.extend(cases)
        except:
            pass

    if options.filter:
        expr = re.compile(options.filter)
        tests = [t for t in tests if expr.search(str(t))]

    tbl = table(", ".join(files))
    if options.columns:
        metrics = [s.strip() for s in options.columns.split(",")]
        metrics = [m for m in metrics if m and not m.endswith("%") and m in metrix_table]
    else:
        metrics = None
    if not metrics:
        metrics = ["name", "samples", "outliers", "min", "median", "gmean", "mean", "stddev"]
    if "name" not in metrics:
        metrics.insert(0, "name")

    for m in metrics:
        if m == "name":
            tbl.newColumn(m, metrix_table[m][0])
        else:
            tbl.newColumn(m, metrix_table[m][0], align = "center")

    needNewRow = True
    for case in sorted(tests, key=lambda x: str(x)):
        if needNewRow:
            tbl.newRow()
            if not options.showall:
                needNewRow = False
        status = case.get("status")
        if status != "run":
            if status != "notrun":
                needNewRow = True
            for m in metrics:
                if m == "name":
                    tbl.newCell(m, str(case))
                else:
                    tbl.newCell(m, status, color = "red")
        else:
            needNewRow = True
            for m in metrics:
                val = metrix_table[m][1](case, None, options.units)
                if isinstance(val, float):
                    tbl.newCell(m, "%.2f %s" % (val, options.units), val)
                else:
                    tbl.newCell(m, val, val)
    if not needNewRow:
        tbl.trimLastRow()

    # output table
    if options.generateHtml:
        if options.format == "moinwiki":
            tbl.htmlPrintTable(sys.stdout, True)
        else:
            htmlPrintHeader(sys.stdout, "Report %s tests from %s" % (len(tests), ", ".join(files)))
            tbl.htmlPrintTable(sys.stdout)
            htmlPrintFooter(sys.stdout)
    else:
        tbl.consolePrintTable(sys.stdout)
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

### Functions and Methods

- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `testlog_parser`
- `print_function`
- `OptionParser`
- `optparse`
- `__future__`
- `table_formatter`


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

