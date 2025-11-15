# Documentation for `docs/samples/winrt/FaceDetection/FaceDetection/MainPage.xaml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/winrt/FaceDetection/FaceDetection/MainPage.xaml_docs.md`
- **File Name**: `MainPage.xaml_docs.md`
- **File Size**: 1,977 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/winrt/FaceDetection/FaceDetection/MainPage.xaml_docs.md](../../../../../docs/samples/winrt/FaceDetection/FaceDetection/MainPage.xaml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/winrt/FaceDetection/FaceDetection/MainPage.xaml`

## File Metadata

- **Full Path**: `samples/winrt/FaceDetection/FaceDetection/MainPage.xaml`
- **File Name**: `MainPage.xaml`
- **File Size**: 1,298 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/winrt/FaceDetection/FaceDetection/MainPage.xaml](../../../../samples/winrt/FaceDetection/FaceDetection/MainPage.xaml)

## Purpose and Role

This file is located in the `samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<Page x:Class="FaceDetection.MainPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
      xmlns:local="using:FaceDetection"
      xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
      mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Button x:Name="InitBtn"
                Width="218"
                Height="67"
                Margin="69,81,0,0"
                HorizontalAlignment="Left"
                VerticalAlignment="Top"
                Click="InitBtn_Click"
                Content="Initialize" />
        <Button x:Name="detectBtn"
                Width="218"
                Height="67"
                Margin="69,168,0,0"
                HorizontalAlignment="Left"
                VerticalAlignment="Top"
                Click="detectBtn_Click"
                Content="Detect Faces" />
        <StackPanel x:Name="cvContainer"
                    Width="883"
                    Height="446"
                    Margin="354,84,0,0"
                    HorizontalAlignment="Left"
                    VerticalAlignment="Top" />

    </Grid>
</Page>

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

