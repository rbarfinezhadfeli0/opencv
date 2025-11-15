# Documentation for `docs/platforms/maven/opencv/scripts/functions_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/maven/opencv/scripts/functions_docs.md`
- **File Name**: `functions_docs.md`
- **File Size**: 1,414 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/maven/opencv/scripts/functions_docs.md](../../../../../docs/platforms/maven/opencv/scripts/functions_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/maven/opencv/scripts` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/maven/opencv/scripts/functions`

## File Metadata

- **Full Path**: `platforms/maven/opencv/scripts/functions`
- **File Name**: `functions`
- **File Size**: 805 bytes
- **File Type**: no extension
- **Link to Source**: [platforms/maven/opencv/scripts/functions](../../../../platforms/maven/opencv/scripts/functions)

## Purpose and Role

This file is located in the `platforms/maven/opencv/scripts` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash
###############################################################
#
# Defines some common functions.
#
# Kerry Billingham <contact [At] AvionicEngineers.{com]>
#
##############################################################
majorHashDefine="#define CV_VERSION_MAJOR"
minorHashDefine="#define CV_VERSION_MINOR"
revisionHashDefine="#define CV_VERSION_REVISION"
statusHashDefine="#define CV_VERSION_STATUS"
versionHeader="../../../../modules/core/include/opencv2/core/version.hpp"

function extract_version() {
    minorVersion=$(grep "${minorHashDefine}" $versionHeader | grep -o ".$")
    majorVersion=$(grep "${majorHashDefine}" $versionHeader | grep -o ".$")
    revision=$(grep "${revisionHashDefine}" $versionHeader | grep -o ".$")

    REPLY="${majorVersion}.${minorVersion}.${revision}"
}

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

