# Documentation for `modules/ts/misc/trace_profiler.py`

## File Metadata

- **Full Path**: `modules/ts/misc/trace_profiler.py`
- **File Name**: `trace_profiler.py`
- **File Size**: 18,508 bytes
- **File Type**: .py
- **Link to Source**: [modules/ts/misc/trace_profiler.py](../../../modules/ts/misc/trace_profiler.py)

## Purpose and Role

This file is located in the `modules/ts/misc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
""" Parse OpenCV trace logs and present summarized statistics in a table

To collect trace logs use OpenCV built with tracing support (enabled by default), set
`OPENCV_TRACE=1` environment variable and run your application. `OpenCVTrace.txt` file will be
created in the current folder.
See https://github.com/opencv/opencv/wiki/Profiling-OpenCV-Applications for more details.

### Options

./trace_profiler.py <TraceLogFile> <num>

<TraceLogFile>  - usually OpenCVTrace.txt
<num>           - number of functions to show (depth)

### Example

./trace_profiler.py OpenCVTrace.txt 2

 ID name                                               count thr         min   ...
                                                                        t-min  ...
  1 main#test_main.cpp:6                                   1   1       88.484  ...
                                                                      200.210  ...

  2 UMatBasicTests_copyTo#test_umat.cpp:176|main          40   1        0.125  ...
                                                                        0.173  ...
"""

from __future__ import print_function

import os
import sys
import csv
from pprint import pprint
from collections import deque

try:
    long        # Python 2
except NameError:
    long = int  # Python 3

# trace.hpp
REGION_FLAG_IMPL_MASK = 15 << 16
REGION_FLAG_IMPL_IPP = 1 << 16
REGION_FLAG_IMPL_OPENCL = 2 << 16

DEBUG = False

if DEBUG:
    dprint = print
    dpprint = pprint
else:
    def dprint(args, **kwargs):
        pass
    def dpprint(args, **kwargs):
        pass

def tryNum(s):
    if s.startswith('0x'):
        try:
            return int(s, 16)
        except ValueError:
            pass
    try:
        return int(s)
    except ValueError:
        pass
    if sys.version_info[0] < 3:
        try:
            return long(s)
        except ValueError:
            pass
    return s

def formatTimestamp(t):
    return "%.3f" % (t * 1e-6)

try:
    from statistics import median
except ImportError:
    def median(lst):
        sortedLst = sorted(lst)
        lstLen = len(lst)
        index = (lstLen - 1) // 2
        if (lstLen % 2):
            return sortedLst[index]
        else:
            return (sortedLst[index] + sortedLst[index + 1]) * 0.5

def getCXXFunctionName(spec):
    def dropParams(spec):
        pos = len(spec) - 1
        depth = 0
        while pos >= 0:
            if spec[pos] == ')':
                depth = depth + 1
            elif spec[pos] == '(':
                depth = depth - 1
                if depth == 0:
                    if pos == 0 or spec[pos - 1] in ['#', ':']:
                        res = dropParams(spec[pos+1:-1])
                        return (spec[:pos] + res[0], res[1])
                    return (spec[:pos], spec[pos:])
            pos = pos - 1
        return (spec, '')

    def extractName(spec):
        pos = len(spec) - 1
        inName = False
        while pos >= 0:
            if spec[pos] == ' ':
                if inName:
                    return spec[pos+1:]
            elif spec[pos].isalnum():
                inName = True
            pos = pos - 1
        return spec

    if spec.startswith('IPP') or spec.startswith('OpenCL'):
        prefix_size = len('IPP') if spec.startswith('IPP') else len('OpenCL')
        prefix = spec[:prefix_size]
        if prefix_size < len(spec) and spec[prefix_size] in ['#', ':']:
            prefix = prefix + spec[prefix_size]
            prefix_size = prefix_size + 1
        begin = prefix_size
        while begin < len(spec):
            if spec[begin].isalnum() or spec[begin] in ['_', ':']:
                break
            begin = begin + 1
        if begin == len(spec):
            return spec
        end = begin
        while end < len(spec):
            if not (spec[end].isalnum() or spec[end] in ['_', ':']):
                break
            end = end + 1
        return prefix + spec[begin:end]

    spec = spec.replace(') const', ')') # const methods
    (ret_type_name, params) = dropParams(spec)
    name = extractName(ret_type_name)
    if 'operator' in name:
        return name + params
    if name.startswith('&'):
        return name[1:]
    return name

stack_size = 10

class Trace:
    def __init__(self, filename=None):
        self.tasks = {}
        self.tasks_list = []
        self.locations = {}
        self.threads_stack = {}
        self.pending_files = deque()
        if filename:
            self.load(filename)

    class TraceTask:
        def __init__(self, threadID, taskID, locationID, beginTimestamp):
            self.threadID = threadID
            self.taskID = taskID
            self.locationID = locationID
            self.beginTimestamp = beginTimestamp
            self.endTimestamp = None
            self.parentTaskID = None
            self.parentThreadID = None
            self.childTask = []
            self.selfTimeIPP = 0
            self.selfTimeOpenCL = 0
            self.totalTimeIPP = 0
            self.totalTimeOpenCL = 0

        def __repr__(self):
            return "TID={} ID={} loc={} parent={}:{} begin={} end={} IPP={}/{} OpenCL={}/{}".format(
                self.threadID, self.taskID, self.locationID, self.parentThreadID, self.parentTaskID,
                self.beginTimestamp, self.endTimestamp, self.totalTimeIPP, self.selfTimeIPP, self.totalTimeOpenCL, self.selfTimeOpenCL)


    class TraceLocation:
        def __init__(self, locationID, filename, line, name, flags):
            self.locationID = locationID
            self.filename = os.path.split(filename)[1]
            self.line = line
            self.name = getCXXFunctionName(name)
            self.flags = flags

        def __str__(self):
            return "{}#{}:{}".format(self.name, self.filename, self.line)

        def __repr__(self):
            return "ID={} {}:{}:{}".format(self.locationID, self.filename, self.line, self.name)

    def parse_file(self, filename):
        dprint("Process file: '{}'".format(filename))
        with open(filename) as infile:
            for line in infile:
                line = str(line).strip()
                if line[0] == "#":
                    if line.startswith("#thread file:"):
                        name = str(line.split(':', 1)[1]).strip()
                        self.pending_files.append(os.path.join(os.path.split(filename)[0], name))
                    continue
                self.parse_line(line)

    def parse_line(self, line):
        opts = line.split(',')
        dpprint(opts)
        if opts[0] == 'l':
            opts = list(csv.reader([line]))[0]  # process quote more
            locationID = int(opts[1])
            filename = str(opts[2])
            line = int(opts[3])
            name = opts[4]
            flags = tryNum(opts[5])
            self.locations[locationID] = self.TraceLocation(locationID, filename, line, name, flags)
            return
        extra_opts = {}
        for e in opts[5:]:
            if not '=' in e:
                continue
            (k, v) = e.split('=')
            extra_opts[k] = tryNum(v)
        if extra_opts:
            dpprint(extra_opts)
        threadID = None
        taskID = None
        locationID = None
        ts = None
        if opts[0] in ['b', 'e']:
            threadID = int(opts[1])
            taskID = int(opts[4])
            locationID = int(opts[3])
            ts = tryNum(opts[2])
        thread_stack = None
        currentTask = (None, None)
        if threadID is not None:
            if not threadID in self.threads_stack:
                thread_stack = deque()
                self.threads_stack[threadID] = thread_stack
            else:
                thread_stack = self.threads_stack[threadID]
            currentTask = None if not thread_stack else thread_stack[-1]
        t = (threadID, taskID)
        if opts[0] == 'b':
            assert not t in self.tasks, "Duplicate task: " + str(t) + repr(self.tasks[t])
            task = self.TraceTask(threadID, taskID, locationID, ts)
            self.tasks[t] = task
            self.tasks_list.append(task)
            thread_stack.append((threadID, taskID))
            if currentTask:
                task.parentThreadID = currentTask[0]
                task.parentTaskID = currentTask[1]
            if 'parentThread' in extra_opts:
                task.parentThreadID = extra_opts['parentThread']
            if 'parent' in extra_opts:
                task.parentTaskID = extra_opts['parent']
        if opts[0] == 'e':
            task = self.tasks[t]
            task.endTimestamp = ts
            if 'tIPP' in extra_opts:
                task.selfTimeIPP = extra_opts['tIPP']
            if 'tOCL' in extra_opts:
                task.selfTimeOpenCL = extra_opts['tOCL']
            thread_stack.pop()

    def load(self, filename):
        self.pending_files.append(filename)
        if DEBUG:
            with open(filename, 'r') as f:
                print(f.read(), end='')
        while self.pending_files:
            self.parse_file(self.pending_files.pop())

    def getParentTask(self, task):
        return self.tasks.get((task.parentThreadID, task.parentTaskID), None)

    def process(self):
        self.tasks_list.sort(key=lambda x: x.beginTimestamp)

        parallel_for_location = None
        for (id, l) in self.locations.items():
            if l.name == 'parallel_for':
                parallel_for_location = l.locationID
                break

        for task in self.tasks_list:
            try:
                task.duration = task.endTimestamp - task.beginTimestamp
                task.selfDuration = task.duration
            except:
                task.duration = None
                task.selfDuration = None
            task.totalTimeIPP = task.selfTimeIPP
            task.totalTimeOpenCL = task.selfTimeOpenCL

        dpprint(self.tasks)
        dprint("Calculate total times")

        for task in self.tasks_list:
            parentTask = self.getParentTask(task)
            if parentTask:
                parentTask.selfDuration = parentTask.selfDuration - task.duration
                parentTask.childTask.append(task)
                timeIPP = task.selfTimeIPP
                timeOpenCL = task.selfTimeOpenCL
                while parentTask:
                    if parentTask.locationID == parallel_for_location:  # TODO parallel_for
                        break
                    parentLocation = self.locations[parentTask.locationID]
                    if (parentLocation.flags & REGION_FLAG_IMPL_MASK) == REGION_FLAG_IMPL_IPP:
                        parentTask.selfTimeIPP = parentTask.selfTimeIPP - timeIPP
                        timeIPP = 0
                    else:
                        parentTask.totalTimeIPP = parentTask.totalTimeIPP + timeIPP
                    if (parentLocation.flags & REGION_FLAG_IMPL_MASK) == REGION_FLAG_IMPL_OPENCL:
                        parentTask.selfTimeOpenCL = parentTask.selfTimeOpenCL - timeOpenCL
                        timeOpenCL = 0
                    else:
                        parentTask.totalTimeOpenCL = parentTask.totalTimeOpenCL + timeOpenCL
                    parentTask = self.getParentTask(parentTask)

        dpprint(self.tasks)
        dprint("Calculate total times (parallel_for)")

        for task in self.tasks_list:
            if task.locationID == parallel_for_location:
                task.selfDuration = 0
                childDuration = sum([t.duration for t in task.childTask])
                if task.duration == 0 or childDuration == 0:
                    continue
                timeCoef = task.duration / float(childDuration)
                childTimeIPP = sum([t.totalTimeIPP for t in task.childTask])
                childTimeOpenCL = sum([t.totalTimeOpenCL for t in task.childTask])
                if childTimeIPP == 0 and childTimeOpenCL == 0:
                    continue
                timeIPP = childTimeIPP * timeCoef
                timeOpenCL = childTimeOpenCL * timeCoef
                parentTask = task
                while parentTask:
                    parentLocation = self.locations[parentTask.locationID]
                    if (parentLocation.flags & REGION_FLAG_IMPL_MASK) == REGION_FLAG_IMPL_IPP:
                        parentTask.selfTimeIPP = parentTask.selfTimeIPP - timeIPP
                        timeIPP = 0
                    else:
                        parentTask.totalTimeIPP = parentTask.totalTimeIPP + timeIPP
                    if (parentLocation.flags & REGION_FLAG_IMPL_MASK) == REGION_FLAG_IMPL_OPENCL:
                        parentTask.selfTimeOpenCL = parentTask.selfTimeOpenCL - timeOpenCL
                        timeOpenCL = 0
                    else:
                        parentTask.totalTimeOpenCL = parentTask.totalTimeOpenCL + timeOpenCL
                    parentTask = self.getParentTask(parentTask)

        dpprint(self.tasks)
        dprint("Done")

    def dump(self, max_entries):
        assert isinstance(max_entries, int)

        class CallInfo():
            def __init__(self, callID):
                self.callID = callID
                self.totalTimes = []
                self.selfTimes = []
                self.threads = set()
                self.selfTimesIPP = []
                self.selfTimesOpenCL = []
                self.totalTimesIPP = []
                self.totalTimesOpenCL = []

        calls = {}

        for currentTask in self.tasks_list:
            task = currentTask
            callID = []
            for i in range(stack_size):
                callID.append(task.locationID)
                task = self.getParentTask(task)
                if not task:
                    break
            callID = tuple(callID)
            if not callID in calls:
                call = CallInfo(callID)
                calls[callID] = call
            else:
                call = calls[callID]
            call.totalTimes.append(currentTask.duration)
            call.selfTimes.append(currentTask.selfDuration)
            call.threads.add(currentTask.threadID)
            call.selfTimesIPP.append(currentTask.selfTimeIPP)
            call.selfTimesOpenCL.append(currentTask.selfTimeOpenCL)
            call.totalTimesIPP.append(currentTask.totalTimeIPP)
            call.totalTimesOpenCL.append(currentTask.totalTimeOpenCL)

        dpprint(self.tasks)
        dpprint(self.locations)
        dpprint(calls)

        calls_self_sum = {k: sum(v.selfTimes) for (k, v) in calls.items()}
        calls_total_sum = {k: sum(v.totalTimes) for (k, v) in calls.items()}
        calls_median = {k: median(v.selfTimes) for (k, v) in calls.items()}
        calls_sorted = sorted(calls.keys(), key=lambda x: calls_self_sum[x], reverse=True)

        calls_self_sum_IPP = {k: sum(v.selfTimesIPP) for (k, v) in calls.items()}
        calls_total_sum_IPP = {k: sum(v.totalTimesIPP) for (k, v) in calls.items()}

        calls_self_sum_OpenCL = {k: sum(v.selfTimesOpenCL) for (k, v) in calls.items()}
        calls_total_sum_OpenCL = {k: sum(v.totalTimesOpenCL) for (k, v) in calls.items()}

        if max_entries > 0 and len(calls_sorted) > max_entries:
            calls_sorted = calls_sorted[:max_entries]

        def formatPercents(p):
            if p is not None:
                return "{:>3d}".format(int(p*100))
            return ''

        name_width = 70
        timestamp_width = 12
        def fmtTS():
            return '{:>' + str(timestamp_width) + '}'
        fmt = "{:>3} {:<"+str(name_width)+"} {:>8} {:>3}"+((' '+fmtTS())*5)+((' '+fmtTS()+' {:>3}')*2)
        fmt2 = "{:>3} {:<"+str(name_width)+"} {:>8} {:>3}"+((' '+fmtTS())*5)+((' '+fmtTS()+' {:>3}')*2)
        print(fmt.format("ID", "name", "count", "thr", "min", "max", "median", "avg", "*self*", "IPP", "%", "OpenCL", "%"))
        print(fmt2.format("", "", "", "", "t-min", "t-max", "t-median", "t-avg", "total", "t-IPP", "%", "t-OpenCL", "%"))
        for (index, callID) in enumerate(calls_sorted):
            call_self_times = calls[callID].selfTimes
            loc0 = self.locations[callID[0]]
            loc_array = []  # [str(callID)]
            for (i, l) in enumerate(callID):
                loc = self.locations[l]
                loc_array.append(loc.name if i > 0 else str(loc))
            loc_str = '|'.join(loc_array)
            if len(loc_str) > name_width: loc_str = loc_str[:name_width-3]+'...'
            print(fmt.format(index + 1, loc_str, len(call_self_times),
                    len(calls[callID].threads),
                    formatTimestamp(min(call_self_times)),
                    formatTimestamp(max(call_self_times)),
                    formatTimestamp(calls_median[callID]),
                    formatTimestamp(sum(call_self_times)/float(len(call_self_times))),
                    formatTimestamp(sum(call_self_times)),
                    formatTimestamp(calls_self_sum_IPP[callID]),
                    formatPercents(calls_self_sum_IPP[callID] / float(calls_self_sum[callID])) if calls_self_sum[callID] > 0 else formatPercents(None),
                    formatTimestamp(calls_self_sum_OpenCL[callID]),
                    formatPercents(calls_self_sum_OpenCL[callID] / float(calls_self_sum[callID])) if calls_self_sum[callID] > 0 else formatPercents(None),
                ))
            call_total_times = calls[callID].totalTimes
            print(fmt2.format("", "", "", "",
                    formatTimestamp(min(call_total_times)),
                    formatTimestamp(max(call_total_times)),
                    formatTimestamp(median(call_total_times)),
                    formatTimestamp(sum(call_total_times)/float(len(call_total_times))),
                    formatTimestamp(sum(call_total_times)),
                    formatTimestamp(calls_total_sum_IPP[callID]),
                    formatPercents(calls_total_sum_IPP[callID] / float(calls_total_sum[callID])) if calls_total_sum[callID] > 0 else formatPercents(None),
                    formatTimestamp(calls_total_sum_OpenCL[callID]),
                    formatPercents(calls_total_sum_OpenCL[callID] / float(calls_total_sum[callID])) if calls_total_sum[callID] > 0 else formatPercents(None),
                ))
            print()

if __name__ == "__main__":
    tracefile = sys.argv[1] if len(sys.argv) > 1 else 'OpenCVTrace.txt'
    count = int(sys.argv[2]) if len(sys.argv) > 2 else 10
    trace = Trace(tracefile)
    trace.process()
    trace.dump(max_entries = count)
    print("OK")
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

- **CallInfo**: A class/struct defined in this file
- **Trace**: A class/struct defined in this file
- **TraceTask**: A class/struct defined in this file
- **TraceLocation**: A class/struct defined in this file

### Functions and Methods

- **getCXXFunctionName()**: A function/method defined in this file
- **formatPercents()**: A function/method defined in this file
- **extractName()**: A function/method defined in this file
- **median()**: A function/method defined in this file
- **dpprint()**: A function/method defined in this file
- **load()**: A function/method defined in this file
- **process()**: A function/method defined in this file
- **dropParams()**: A function/method defined in this file
- **getParentTask()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file
- **parse_line()**: A function/method defined in this file
- **__str__()**: A function/method defined in this file
- **parse_file()**: A function/method defined in this file
- **dprint()**: A function/method defined in this file
- **__repr__()**: A function/method defined in this file
- **dump()**: A function/method defined in this file
- **tryNum()**: A function/method defined in this file
- **formatTimestamp()**: A function/method defined in this file
- **fmtTS()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `sys`
- `os`
- `csv`
- `deque`
- `print_function`
- `statistics`
- `__future__`
- `collections`
- `median`
- `pprint`


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

