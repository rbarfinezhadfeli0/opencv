# Documentation for `samples/winrt/OcvImageProcessing/OcvImageProcessing/App.xaml`

## File Metadata

- **Full Path**: `samples/winrt/OcvImageProcessing/OcvImageProcessing/App.xaml`
- **File Name**: `App.xaml`
- **File Size**: 738 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/winrt/OcvImageProcessing/OcvImageProcessing/App.xaml](../../../../samples/winrt/OcvImageProcessing/OcvImageProcessing/App.xaml)

## Purpose and Role

This file is located in the `samples/winrt/OcvImageProcessing/OcvImageProcessing` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<Application
    x:Class="OcvImageProcessing.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:OcvImageProcessing">

    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>

                <!--
                    Styles that define common aspects of the platform look and feel
                    Required by Visual Studio project and item templates
                 -->
                <ResourceDictionary Source="Common/StandardStyles.xaml"/>
            </ResourceDictionary.MergedDictionaries>

        </ResourceDictionary>
    </Application.Resources>
</Application>

```

## General Information

This file is part of the OpenCV repository infrastructure.

