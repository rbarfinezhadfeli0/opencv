# Documentation for `platforms/scripts/pylintrc`

## File Metadata

- **Full Path**: `platforms/scripts/pylintrc`
- **File Name**: `pylintrc`
- **File Size**: 568 bytes
- **File Type**: no extension
- **Link to Source**: [platforms/scripts/pylintrc](../../platforms/scripts/pylintrc)

## Purpose and Role

This file is located in the `platforms/scripts` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
[MESSAGES CONTROL]

# Disable all to choose the Tests one by one
disable=all

# Tests
enable=bad-indentation,       # Used when an unexpected number of indentation’s tabulations or spaces has been found.
       mixed-indentation,     # Used when there are some mixed tabs and spaces in a module.
       unnecessary-semicolon, # Used when a statement is ended by a semi-colon (";"), which isn’t necessary.
       unused-variable        # Used when a variable is defined but not used. (Use _var to ignore var).


[REPORTS]

# Activate the evaluation score.
score=no

```

## General Information

This file is part of the OpenCV repository infrastructure.

