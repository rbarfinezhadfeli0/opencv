# Documentation for `modules/objc/generator/templates/objc_module_header.template`

## File Metadata

- **Full Path**: `modules/objc/generator/templates/objc_module_header.template`
- **File Name**: `objc_module_header.template`
- **File Size**: 367 bytes
- **File Type**: .template
- **Link to Source**: [modules/objc/generator/templates/objc_module_header.template](../../../../modules/objc/generator/templates/objc_module_header.template)

## Purpose and Role

This file is located in the `modules/objc/generator/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
//
// This file is auto-generated. Please don't modify it!
//
#pragma once

#ifdef __cplusplus
//#import "opencv.hpp"
$additionalImports
#else
#define CV_EXPORTS
#endif

#import <Foundation/Foundation.h>

$forwardDeclarations

$enumDeclarations

NS_ASSUME_NONNULL_BEGIN

$docs
CV_EXPORTS @interface $objcName : $base

$methodDeclarations

@end

NS_ASSUME_NONNULL_END

```

## General Information

This file is part of the OpenCV repository infrastructure.

