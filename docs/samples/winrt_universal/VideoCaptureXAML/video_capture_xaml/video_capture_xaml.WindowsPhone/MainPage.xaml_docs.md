# Documentation for `samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.WindowsPhone/MainPage.xaml`

## File Metadata

- **Full Path**: `samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.WindowsPhone/MainPage.xaml`
- **File Name**: `MainPage.xaml`
- **File Size**: 1,048 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.WindowsPhone/MainPage.xaml](../../../../../samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.WindowsPhone/MainPage.xaml)

## Purpose and Role

This file is located in the `samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.WindowsPhone` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<Page x:Class="video_capture_xaml.MainPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
      xmlns:local="using:video_capture_xaml"
      xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
      Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
      mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <TextBlock Margin="20,35,0,0"
                   HorizontalAlignment="Left"
                   VerticalAlignment="Top"
                   FontSize="24"
                   TextWrapping="Wrap">
            <Run Text="OpenCV: videoio" />
            <Run />
        </TextBlock>
        <Image Name="cvImage"
               Width="640"
               Height="480"
               Margin="20,100,0,0"
               HorizontalAlignment="Left"
               VerticalAlignment="Top" />
    </Grid>
</Page>
```

## General Information

This file is part of the OpenCV repository infrastructure.

