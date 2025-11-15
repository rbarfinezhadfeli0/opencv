# Documentation for `docs/cmake/templates/setup_vars_macosx.sh.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/setup_vars_macosx.sh.in_docs.md`
- **File Name**: `setup_vars_macosx.sh.in_docs.md`
- **File Size**: 1,323 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/setup_vars_macosx.sh.in_docs.md](../../../docs/cmake/templates/setup_vars_macosx.sh.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/setup_vars_macosx.sh.in`

## File Metadata

- **Full Path**: `cmake/templates/setup_vars_macosx.sh.in`
- **File Name**: `setup_vars_macosx.sh.in`
- **File Size**: 734 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/setup_vars_macosx.sh.in](../../cmake/templates/setup_vars_macosx.sh.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash

SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"

[[ ! "${OPENCV_QUIET}" ]] && ( echo "Setting vars for OpenCV @OPENCV_VERSION@" )
export DYLD_LIBRARY_PATH="$SCRIPT_DIR/@OPENCV_LIB_RUNTIME_DIR_RELATIVE_CMAKECONFIG@:$DYLD_LIBRARY_PATH"

if [[ ! "$OPENCV_SKIP_PYTHON" ]]; then
  PYTHONPATH_OPENCV="$SCRIPT_DIR/@OPENCV_PYTHON_DIR_RELATIVE_CMAKECONFIG@"
  [[ ! "${OPENCV_QUIET}" ]] && ( echo "Append PYTHONPATH: ${PYTHONPATH_OPENCV}" )
  export PYTHONPATH="${PYTHONPATH_OPENCV}:$PYTHONPATH"
fi

# Don't exec in "sourced" mode
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
  if [[ $# -ne 0 ]]; then
    [[ ! "${OPENCV_QUIET}" && "${OPENCV_VERBOSE}" ]] && ( echo "Executing: $*" )
    exec "$@"
  fi
fi

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

