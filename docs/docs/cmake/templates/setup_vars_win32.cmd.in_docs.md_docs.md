# Documentation for `docs/cmake/templates/setup_vars_win32.cmd.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/setup_vars_win32.cmd.in_docs.md`
- **File Name**: `setup_vars_win32.cmd.in_docs.md`
- **File Size**: 1,412 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/setup_vars_win32.cmd.in_docs.md](../../../docs/cmake/templates/setup_vars_win32.cmd.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/setup_vars_win32.cmd.in`

## File Metadata

- **Full Path**: `cmake/templates/setup_vars_win32.cmd.in`
- **File Name**: `setup_vars_win32.cmd.in`
- **File Size**: 823 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/setup_vars_win32.cmd.in](../../cmake/templates/setup_vars_win32.cmd.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
@ECHO OFF

SET "SCRIPT_DIR=%~dp0"

IF NOT DEFINED OPENCV_QUIET ( ECHO Setting vars for OpenCV @OPENCV_VERSION@ )
SET "PATH=%SCRIPT_DIR%\@OPENCV_LIB_RUNTIME_DIR_RELATIVE_CMAKECONFIG@;%PATH%"

IF NOT DEFINED OPENCV_SKIP_PYTHON CALL :SET_PYTHON

SET SCRIPT_DIR=

IF NOT [%1] == [] GOTO :RUN_COMMAND

GOTO :EOF

:RUN_COMMAND
SET RUN_INTERACTIVE=1
echo %CMDCMDLINE% | find /i "%~0" >nul
IF NOT errorlevel 1 set RUN_INTERACTIVE=0

%*
SET RESULT=%ERRORLEVEL%
IF %RESULT% NEQ 0 (
  IF _%RUN_INTERACTIVE%_==_0_ ( IF NOT DEFINED OPENCV_BATCH_MODE ( pause ) )
)
EXIT /B %RESULT%

:SET_PYTHON
SET "PYTHONPATH_OPENCV=%SCRIPT_DIR%\@OPENCV_PYTHON_DIR_RELATIVE_CMAKECONFIG@"
IF NOT DEFINED OPENCV_QUIET ( ECHO Append PYTHONPATH: %PYTHONPATH_OPENCV% )
SET "PYTHONPATH=%PYTHONPATH_OPENCV%;%PYTHONPATH%"
SET PYTHONPATH_OPENCV=
EXIT /B


:EOF

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

