# Documentation for `samples/winrt_universal/PhoneTutorial/MainPage.xaml`

## File Metadata

- **Full Path**: `samples/winrt_universal/PhoneTutorial/MainPage.xaml`
- **File Name**: `MainPage.xaml`
- **File Size**: 769 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/winrt_universal/PhoneTutorial/MainPage.xaml](../../../samples/winrt_universal/PhoneTutorial/MainPage.xaml)

## Purpose and Role

This file is located in the `samples/winrt_universal/PhoneTutorial` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<Page
    x:Class="PhoneTutorial.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:PhoneTutorial"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid>
        <StackPanel>
            <Image x:Name="image" />
            <Button x:Name="Process" Content="Process" HorizontalAlignment="Center" Click="Process_Click"/>
            <Button x:Name="Reset" Content="Reset" HorizontalAlignment="Center" Click="Reset_Click"/>
        </StackPanel>
    </Grid>
</Page>

```

## General Information

This file is part of the OpenCV repository infrastructure.

