# Documentation for `platforms/winrt/setup_winrt.bat`

## File Metadata

- **Full Path**: `platforms/winrt/setup_winrt.bat`
- **File Name**: `setup_winrt.bat`
- **File Size**: 82 bytes
- **File Type**: .bat
- **Link to Source**: [platforms/winrt/setup_winrt.bat](../../platforms/winrt/setup_winrt.bat)

## Purpose and Role

This file is located in the `platforms/winrt` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
@echo off
Powershell.exe -ExecutionPolicy Unrestricted -File setup_winrt.ps1 %*

```

## General Information

This file is part of the OpenCV repository infrastructure.

