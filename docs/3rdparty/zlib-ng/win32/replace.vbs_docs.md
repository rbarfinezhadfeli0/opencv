# Documentation for `3rdparty/zlib-ng/win32/replace.vbs`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/win32/replace.vbs`
- **File Name**: `replace.vbs`
- **File Size**: 458 bytes
- **File Type**: .vbs
- **Link to Source**: [3rdparty/zlib-ng/win32/replace.vbs](../../../3rdparty/zlib-ng/win32/replace.vbs)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/win32` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
strInputFileName = Wscript.Arguments(0)
strOutputFileName = Wscript.Arguments(1)
strOldText = Wscript.Arguments(2)
strNewText = Wscript.Arguments(3)

Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objFile = objFSO.OpenTextFile(strInputFileName, 1)

strText = objFile.ReadAll
objFile.Close
strNewText = Replace(strText, strOldText, strNewText)

Set objFile = objFSO.OpenTextFile(strOutputFileName, 2, True)
objFile.Write strNewText
objFile.Close

```

## General Information

This file is part of the OpenCV repository infrastructure.

