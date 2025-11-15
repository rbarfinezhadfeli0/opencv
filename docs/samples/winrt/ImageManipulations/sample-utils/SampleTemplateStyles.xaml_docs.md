# Documentation for `samples/winrt/ImageManipulations/sample-utils/SampleTemplateStyles.xaml`

## File Metadata

- **Full Path**: `samples/winrt/ImageManipulations/sample-utils/SampleTemplateStyles.xaml`
- **File Name**: `SampleTemplateStyles.xaml`
- **File Size**: 2,195 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/winrt/ImageManipulations/sample-utils/SampleTemplateStyles.xaml](../../../../samples/winrt/ImageManipulations/sample-utils/SampleTemplateStyles.xaml)

## Purpose and Role

This file is located in the `samples/winrt/ImageManipulations/sample-utils` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<!--
//*********************************************************
//
// Copyright (c) Microsoft. All rights reserved.
// THIS CODE IS PROVIDED *AS IS* WITHOUT WARRANTY OF
// ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING ANY
// IMPLIED WARRANTIES OF FITNESS FOR A PARTICULAR
// PURPOSE, MERCHANTABILITY, OR NON-INFRINGEMENT.
//
//*********************************************************
-->
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">

    <Style x:Key="TitleTextStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI Light" />
        <Setter Property="FontSize" Value="16" />
    </Style>
    <Style x:Key="HeaderTextStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI Semilight" />
        <Setter Property="FontSize" Value="26.667" />
        <Setter Property="Margin" Value="0,0,0,25" />
    </Style>
    <Style x:Key="H2Style" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI" />
        <Setter Property="FontSize" Value="14.667" />
        <Setter Property="Margin" Value="0,0,0,0" />
    </Style>
    <Style x:Key="SubheaderTextStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI Semilight" />
        <Setter Property="FontSize" Value="14.667" />
        <Setter Property="Margin" Value="0,0,0,5" />
    </Style>
    <Style x:Key="BasicTextStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI Light" />
        <Setter Property="FontSize" Value="16" />
    </Style>
    <Style x:Key="SeparatorStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI" />
        <Setter Property="FontSize" Value="9" />
    </Style>
    <Style x:Key="FooterStyle" TargetType="TextBlock">
        <Setter Property="FontFamily" Value="Segoe UI" />
        <Setter Property="FontSize" Value="12" />
        <Setter Property="Margin" Value="0,8,0,0" />
    </Style>
    <Style x:Key="HyperlinkStyle" TargetType="HyperlinkButton">
        <Setter Property="Padding" Value="5"/>
    </Style>
</ResourceDictionary>

```

## General Information

This file is part of the OpenCV repository infrastructure.

