# Documentation for `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml_docs.md`
- **File Name**: `App.xaml_docs.md`
- **File Size**: 1,765 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml_docs.md](../../../../../../docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml`

## File Metadata

- **Full Path**: `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml`
- **File Name**: `App.xaml`
- **File Size**: 935 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml](../../../../../samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/App.xaml)

## Purpose and Role

This file is located in the `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<Application
    x:Class="PhoneXamlDirect3DApp1.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
    xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone">

    <!--Application Resources-->
    <Application.Resources>
        <local:LocalizedStrings xmlns:local="clr-namespace:PhoneXamlDirect3DApp1" x:Key="LocalizedStrings"/>
    </Application.Resources>

    <Application.ApplicationLifetimeObjects>
        <!--Required object that handles lifetime events for the application-->
        <shell:PhoneApplicationService
            Launching="Application_Launching" Closing="Application_Closing"
            Activated="Application_Activated" Deactivated="Application_Deactivated"/>
    </Application.ApplicationLifetimeObjects>

</Application>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

