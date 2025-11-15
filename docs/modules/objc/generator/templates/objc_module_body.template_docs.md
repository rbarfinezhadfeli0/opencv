# Documentation for `modules/objc/generator/templates/objc_module_body.template`

## File Metadata

- **Full Path**: `modules/objc/generator/templates/objc_module_body.template`
- **File Name**: `objc_module_body.template`
- **File Size**: 175 bytes
- **File Type**: .template
- **Link to Source**: [modules/objc/generator/templates/objc_module_body.template](../../../../modules/objc/generator/templates/objc_module_body.template)

## Purpose and Role

This file is located in the `modules/objc/generator/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
//
// This file is auto-generated. Please don't modify it!
//

#import "$objcName.h"
#import "CVObjcUtil.h"

$imports

@implementation $objcName

$methodImplementations

@end

```

## General Information

This file is part of the OpenCV repository infrastructure.

