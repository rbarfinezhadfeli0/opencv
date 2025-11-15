# Documentation for `modules/core/src/opencl/runtime/generator/generate.sh`

## File Metadata

- **Full Path**: `modules/core/src/opencl/runtime/generator/generate.sh`
- **File Name**: `generate.sh`
- **File Size**: 293 bytes
- **File Type**: .sh
- **Link to Source**: [modules/core/src/opencl/runtime/generator/generate.sh](../../../../../../modules/core/src/opencl/runtime/generator/generate.sh)

## Purpose and Role

This file is located in the `modules/core/src/opencl/runtime/generator` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash -e
echo "Generate files for CL runtime..."
python parser_cl.py opencl_core < sources/cl.h
python parser_cl.py opencl_gl < sources/cl_gl.h

python parser_clamdblas.py < sources/clAmdBlas.h
python parser_clamdfft.py < sources/clAmdFft.h

echo "Generate files for CL runtime... Done"

```

## General Information

This file is part of the OpenCV repository infrastructure.

