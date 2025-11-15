# Documentation for `modules/ts/misc/perf_tests_timing.py`

## File Metadata

- **Full Path**: `modules/ts/misc/perf_tests_timing.py`
- **File Name**: `perf_tests_timing.py`
- **File Size**: 7,374 bytes
- **File Type**: .py
- **Link to Source**: [modules/ts/misc/perf_tests_timing.py](../../../modules/ts/misc/perf_tests_timing.py)

## Purpose and Role

This file is located in the `modules/ts/misc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
""" Prints total execution time and number of total/failed tests.

Performance data is stored in the GTest log file created by performance tests. Default name is
`test_details.xml`. It can be changed with the `--gtest_output=xml:<location>/<filename>.xml` test
option. See https://github.com/opencv/opencv/wiki/HowToUsePerfTests for more details.

This script uses XML test log to produce basic runtime statistics in a text or HTML table.

### Example:

./perf_tests_timing.py  opencv_perf_core.xml

Overall time: 222.71 min

Module                       Testsuit                        Time (min) Num of tests Failed
opencv Gemm::OCL_GemmFixture                                  113.669        24
opencv dft::Size_MatType_FlagsType_NzeroRows                   21.127       180
opencv Dft::OCL_DftFixture                                     11.153       144
opencv convertTo::Size_DepthSrc_DepthDst_Channels_alpha        7.992        392
opencv Normalize::OCL_NormalizeFixture                         5.412         96
...    ...                                                      ...         ...
"""

from __future__ import print_function
import testlog_parser, sys, os, xml, glob, re
from table_formatter import *
from optparse import OptionParser
from operator import itemgetter, attrgetter
from summary import getSetName, alphanum_keyselector
import re

if __name__ == "__main__":
    usage = "%prog <log_name>.xml [...]"
    parser = OptionParser(usage = usage)

    parser.add_option("-o", "--output", dest = "format",
        help = "output results in text format (can be 'txt', 'html' or 'auto' - default)",
        metavar = 'FMT', default = 'auto')

    parser.add_option("--failed-only", action = "store_true", dest = "failedOnly",
        help = "print only failed tests", default = False)

    (options, args) = parser.parse_args()

    options.generateHtml = detectHtmlOutputType(options.format)

    files = []
    testsuits = [] # testsuit module, name, time, num, flag for failed tests
    overall_time = 0

    seen = set()
    for arg in args:
        if ("*" in arg) or ("?" in arg):
            flist = [os.path.abspath(f) for f in glob.glob(arg)]
            flist = sorted(flist, key= lambda text: str(text).replace("M", "_"))
            files.extend([ x for x in flist if x not in seen and not seen.add(x)])
        else:
            fname = os.path.abspath(arg)
            if fname not in seen and not seen.add(fname):
                files.append(fname)

            file = os.path.abspath(fname)
            if not os.path.isfile(file):
                sys.stderr.write("IOError reading \"" + file + "\" - " + str(err) + os.linesep)
                parser.print_help()
                exit(0)

            fname = os.path.basename(fname)
            find_module_name = re.search(r'([^_]*)', fname)
            module_name = find_module_name.group(0)

            test_sets = []
            try:
                tests = testlog_parser.parseLogFile(file)
                if tests:
                    test_sets.append((os.path.basename(file), tests))
            except IOError as err:
                sys.stderr.write("IOError reading \"" + file + "\" - " + str(err) + os.linesep)
            except xml.parsers.expat.ExpatError as err:
                sys.stderr.write("ExpatError reading \"" + file + "\" - " + str(err) + os.linesep)

            if not test_sets:
                continue

            # find matches
            setsCount = len(test_sets)
            test_cases = {}

            name_extractor = lambda name: str(name)

            for i in range(setsCount):
                for case in test_sets[i][1]:
                    name = name_extractor(case)
                    if name not in test_cases:
                        test_cases[name] = [None] * setsCount
                    test_cases[name][i] = case

            prevGroupName = None
            suit_time = 0
            suit_num = 0
            fails_num = 0
            for name in sorted(test_cases.keys(), key=alphanum_keyselector):
                cases = test_cases[name]

                groupName = next(c for c in cases if c).shortName()
                if groupName != prevGroupName:
                    if prevGroupName != None:
                        suit_time = suit_time/60 #from seconds to minutes
                        testsuits.append({'module': module_name, 'name': prevGroupName, \
                            'time': suit_time, 'num': suit_num, 'failed': fails_num})
                        overall_time += suit_time
                        suit_time = 0
                        suit_num = 0
                        fails_num = 0
                    prevGroupName = groupName

                for i in range(setsCount):
                    case = cases[i]
                    if not case is None:
                        suit_num += 1
                        if case.get('status') == 'run':
                            suit_time += case.get('time')
                        if case.get('status') == 'failed':
                            fails_num += 1

            # last testsuit processing
            suit_time = suit_time/60
            testsuits.append({'module': module_name, 'name': prevGroupName, \
                'time': suit_time, 'num': suit_num, 'failed': fails_num})
            overall_time += suit_time

    if len(testsuits)==0:
        exit(0)

    tbl = table()
    rows = 0

    if not options.failedOnly:
        tbl.newColumn('module', 'Module', align = 'left', cssclass = 'col_name')
        tbl.newColumn('name', 'Testsuit', align = 'left', cssclass = 'col_name')
        tbl.newColumn('time', 'Time (min)', align = 'center', cssclass = 'col_name')
        tbl.newColumn('num', 'Num of tests', align = 'center', cssclass = 'col_name')
        tbl.newColumn('failed', 'Failed', align = 'center', cssclass = 'col_name')

        # rows
        for suit in sorted(testsuits, key = lambda suit: suit['time'], reverse = True):
            tbl.newRow()
            tbl.newCell('module', suit['module'])
            tbl.newCell('name', suit['name'])
            tbl.newCell('time', formatValue(suit['time'], '', ''), suit['time'])
            tbl.newCell('num', suit['num'])
            if (suit['failed'] != 0):
                tbl.newCell('failed', suit['failed'])
            else:
                tbl.newCell('failed', ' ')
            rows += 1

    else:
        tbl.newColumn('module', 'Module', align = 'left', cssclass = 'col_name')
        tbl.newColumn('name', 'Testsuit', align = 'left', cssclass = 'col_name')
        tbl.newColumn('failed', 'Failed', align = 'center', cssclass = 'col_name')

        # rows
        for suit in sorted(testsuits, key = lambda suit: suit['time'], reverse = True):
            if (suit['failed'] != 0):
                tbl.newRow()
                tbl.newCell('module', suit['module'])
                tbl.newCell('name', suit['name'])
                tbl.newCell('failed', suit['failed'])
                rows += 1

    # output table
    if rows:
        if options.generateHtml:
            tbl.htmlPrintTable(sys.stdout)
            htmlPrintFooter(sys.stdout)
        else:
            if not options.failedOnly:
                print('\nOverall time: %.2f min\n' % overall_time)
            tbl.consolePrintTable(sys.stdout)
            print(2 * '\n')
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
- `getSetName`
- `re`
- `print_function`
- `OptionParser`
- `operator`
- `optparse`
- `__future__`
- `table_formatter`
- `seconds`
- `summary`
- `itemgetter`


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

