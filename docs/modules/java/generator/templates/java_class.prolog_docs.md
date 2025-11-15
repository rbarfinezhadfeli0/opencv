# Documentation for `modules/java/generator/templates/java_class.prolog`

## File Metadata

- **Full Path**: `modules/java/generator/templates/java_class.prolog`
- **File Name**: `java_class.prolog`
- **File Size**: 394 bytes
- **File Type**: .prolog
- **Link to Source**: [modules/java/generator/templates/java_class.prolog](../../../../modules/java/generator/templates/java_class.prolog)

## Purpose and Role

This file is located in the `modules/java/generator/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
//
// This file is auto-generated. Please don't modify it!
//
package org.opencv.$module;

$imports

$docs$annotation
public class $jname {

    protected final long nativeObj;
    protected $jname(long addr) { nativeObj = addr; }

    public long getNativeObjAddr() { return nativeObj; }

    // internal usage only
    public static $jname __fromPtr__(long addr) { return new $jname(addr); }

```

## General Information

This file is part of the OpenCV repository infrastructure.

