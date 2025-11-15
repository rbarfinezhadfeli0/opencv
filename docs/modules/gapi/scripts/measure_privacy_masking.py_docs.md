# Documentation for `modules/gapi/scripts/measure_privacy_masking.py`

## File Metadata

- **Full Path**: `modules/gapi/scripts/measure_privacy_masking.py`
- **File Name**: `measure_privacy_masking.py`
- **File Size**: 3,569 bytes
- **File Type**: .py
- **Link to Source**: [modules/gapi/scripts/measure_privacy_masking.py](../../../modules/gapi/scripts/measure_privacy_masking.py)

## Purpose and Role

This file is located in the `modules/gapi/scripts` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python3

import sys
import subprocess
import re
from enum import Enum

## Helper functions ##################################################
##
def fmt_bool(x):
    return ("true" if x else "false")

def fmt_bin(base, prec, model):
    return "%s/%s/%s/%s.xml" % (base, model, prec, model)

## The script itself #################################################
##
if len(sys.argv) != 3:
    print("Usage: %s /path/to/input/video /path/to/models" % sys.argv[0])
    exit(1)

input_file_path   = sys.argv[1]
intel_models_path = sys.argv[2]

app             = "bin/example_gapi_privacy_masking_camera"
intel_fd_model  = "face-detection-retail-0005"
intel_lpd_model = "vehicle-license-plate-detection-barrier-0106"
output_file     = "out_results.csv"

tgts = [ ("CPU", "INT8")
       , ("CPU", "FP32")
       , ("GPU", "FP16")
       ]

class Policy(Enum):
    Traditional = 1
    Streaming   = 2

# From mode to cmd arg
mods = [ (Policy.Traditional, True)
       , (Policy.Streaming,  False)
       ]

class UI(Enum):
    With    = 1
    Without = 2

# From mode to cmd arg
ui = [ (UI.With,   False)
     , (UI.Without, True)
     ]

fd_fmt_bin  = lambda prec : fmt_bin(intel_models_path, prec, intel_fd_model)
lpd_fmt_bin = lambda prec : fmt_bin(intel_models_path, prec, intel_lpd_model)

# Performance comparison table
table={}

# Collect the performance data
for m in mods:              # Execution mode (trad/stream)
    for u in ui:            # UI mode (on/off)
        for f in tgts:      # FD model
            for p in tgts:  # LPD model
                cmd = [ app
                      , ("--input=%s"  % input_file_path)   # input file
                      , ("--faced=%s"  % f[0])              # FD device target
                      , ("--facem=%s"  % fd_fmt_bin(f[1]))  # FD model @ precision
                      , ("--platd=%s"  % p[0])              # LPD device target
                      , ("--platm=%s"  % lpd_fmt_bin(p[1])) # LPD model @ precision
                      , ("--trad=%s"   % fmt_bool(m[1]))    # Execution policy
                      , ("--noshow=%s" % fmt_bool(u[1]))    # UI mode (show/no show)
                      ]
                out = str(subprocess.check_output(cmd))
                match = re.search('Processed [0-9]+ frames \(([0-9]+\.[0-9]+) FPS\)', out)
                fps = float(match.group(1))
                print(cmd, fps, "FPS")
                table[m[0],u[0],f,p] = fps

# Write the performance summary
# Columns: all other components (mode, ui)
with open(output_file, 'w') as csv:
    # CSV header
    csv.write("FD,LPD,Serial(UI),Serial(no-UI),Streaming(UI),Streaming(no-UI),Effect(UI),Effect(no-UI)\n")

    for f in tgts:      # FD model
        for p in tgts:  # LPD model
            row = "%s/%s,%s/%s" % (f[0], f[1], p[0], p[1])                # FD precision, LPD precision
            row += ",%f"    % table[Policy.Traditional,UI.With,   f,p]    # Serial/UI
            row += ",%f"    % table[Policy.Traditional,UI.Without,f,p]    # Serial/no UI
            row += ",%f"    % table[Policy.Streaming,  UI.With,   f,p]    # Streaming/UI
            row += ",%f"    % table[Policy.Streaming,  UI.Without,f,p]    # Streaming/no UI

            effect_ui   = table[Policy.Streaming,UI.With,   f,p] / table[Policy.Traditional,UI.With,   f,p]
            effect_noui = table[Policy.Streaming,UI.Without,f,p] / table[Policy.Traditional,UI.Without,f,p]
            row += ",%f,%f" % (effect_ui,effect_noui)
            row += "\n"
            csv.write(row)

print("DONE: ", output_file)
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

- **UI**: A class/struct defined in this file
- **Policy**: A class/struct defined in this file

### Functions and Methods

- **fmt_bool()**: A function/method defined in this file
- **fmt_bin()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `sys`
- `enum`
- `re`
- `subprocess`
- `Enum`


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

