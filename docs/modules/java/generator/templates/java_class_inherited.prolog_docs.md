# Documentation for `modules/java/generator/templates/java_class_inherited.prolog`

## File Metadata

- **Full Path**: `modules/java/generator/templates/java_class_inherited.prolog`
- **File Name**: `java_class_inherited.prolog`
- **File Size**: 309 bytes
- **File Type**: .prolog
- **Link to Source**: [modules/java/generator/templates/java_class_inherited.prolog](../../../../modules/java/generator/templates/java_class_inherited.prolog)

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
public class $jname extends $base {

    protected $jname(long addr) { super(addr); }

    // internal usage only
    public static $jname __fromPtr__(long addr) { return new $jname(addr); }

```

## General Information

This file is part of the OpenCV repository infrastructure.

