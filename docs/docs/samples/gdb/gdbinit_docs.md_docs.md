# Documentation for `docs/samples/gdb/gdbinit_docs.md`

## File Metadata

- **Full Path**: `docs/samples/gdb/gdbinit_docs.md`
- **File Name**: `gdbinit_docs.md`
- **File Size**: 1,163 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/gdb/gdbinit_docs.md](../../../docs/samples/gdb/gdbinit_docs.md)

## Purpose and Role

This file is located in the `docs/samples/gdb` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/gdb/gdbinit`

## File Metadata

- **Full Path**: `samples/gdb/gdbinit`
- **File Name**: `gdbinit`
- **File Size**: 665 bytes
- **File Type**: no extension
- **Link to Source**: [samples/gdb/gdbinit](../../samples/gdb/gdbinit)

## Purpose and Role

This file is located in the `samples/gdb` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
set auto-load local-gdbinit on
set print elements 0
add-auto-load-safe-path /

python
# Update GDB's Python paths with the `sys.path` values of the local
#  Python installation, whether that is brew'ed Python, a virtualenv,
#  or another system python.

# Convert GDB to interpret in Python

import os, subprocess, sys

# Execute a Python using the user's shell and pull out the sys.path (for site-packages)
paths = subprocess.check_output('/usr/bin/python3 -c "import os,sys;print(os.linesep.join(sys.path).strip())"',shell=True).decode("utf-8").split()

# Extend GDB's Python's search path
sys.path.extend(paths)

end


source /your/path/to/mat_pretty_printer.py

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

