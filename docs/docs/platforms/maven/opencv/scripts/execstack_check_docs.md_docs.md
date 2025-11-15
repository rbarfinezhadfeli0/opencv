# Documentation for `docs/platforms/maven/opencv/scripts/execstack_check_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/maven/opencv/scripts/execstack_check_docs.md`
- **File Name**: `execstack_check_docs.md`
- **File Size**: 1,578 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/maven/opencv/scripts/execstack_check_docs.md](../../../../../docs/platforms/maven/opencv/scripts/execstack_check_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/maven/opencv/scripts` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/maven/opencv/scripts/execstack_check`

## File Metadata

- **Full Path**: `platforms/maven/opencv/scripts/execstack_check`
- **File Name**: `execstack_check`
- **File Size**: 939 bytes
- **File Type**: no extension
- **Link to Source**: [platforms/maven/opencv/scripts/execstack_check](../../../../platforms/maven/opencv/scripts/execstack_check)

## Purpose and Role

This file is located in the `platforms/maven/opencv/scripts` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash

##################################################################
#
#	This script will clear the executable flag on
#	the specified library and then check it has
#	been cleared as a separate operation.
#
#	$1 - The absolute path to the OpenCV native library.
#
#	Returns:
#	0 - The executable flag has been cleared
#	1 - The executable flag could NOT be cleared (failure).
#
#   Kerry Billingham <contact (at) avionicengineers (d0t) com>
#   11 March 2017
#
##################################################################
red=$'\e[1;31m'
green=$'\e[1;32m'
end=$'\e[0m'
echo "${green}[INFO] Checking that the native library executable stack flag is NOT set.${end}"
BINARY=execstack
$BINARY --help > /dev/null || BINARY=/usr/sbin/execstack
$BINARY -c $1
$BINARY -q $1 | grep -o ^-
if [ $? -ne 0 ]; then
    echo
    echo "${red}[ERROR] The Executable Flag could not be cleared on the library $1.${end}"
    exit 1
fi
exit 0

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

