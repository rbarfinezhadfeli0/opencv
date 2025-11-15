# Documentation for `docs/3rdparty/zlib-ng/win32/replace.vbs_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/win32/replace.vbs_docs.md`
- **File Name**: `replace.vbs_docs.md`
- **File Size**: 1,026 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/win32/replace.vbs_docs.md](../../../../docs/3rdparty/zlib-ng/win32/replace.vbs_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng/win32` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

