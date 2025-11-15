# Documentation for `docs/samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml_docs.md`
- **File Name**: `MainPage.xaml_docs.md`
- **File Size**: 3,019 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml_docs.md](../../../../../../docs/samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml`

## File Metadata

- **Full Path**: `samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml`
- **File Name**: `MainPage.xaml`
- **File Size**: 2,187 bytes
- **File Type**: .xaml
- **Link to Source**: [samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml](../../../../../samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/MainPage.xaml)

## Purpose and Role

This file is located in the `samples/wp8/OcvRotatingCube/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<phone:PhoneApplicationPage x:Class="PhoneXamlDirect3DApp1.MainPage"
                            xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                            xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
                            xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
                            xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
                            xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
                            FontFamily="{StaticResource PhoneFontFamilyNormal}"
                            FontSize="{StaticResource PhoneFontSizeNormal}"
                            Foreground="{StaticResource PhoneForegroundBrush}"
                            Orientation="Portrait"
                            SupportedOrientations="Portrait"
                            shell:SystemTray.IsVisible="True"
                            mc:Ignorable="d">

    <!--  LayoutRoot is the root grid where all page content is placed  -->
    <Grid x:Name="LayoutRoot" Background="Transparent">
        <DrawingSurface x:Name="DrawingSurface" Loaded="DrawingSurface_Loaded" />
        <StackPanel Margin="40">
            <RadioButton x:Name="Normal"
                         Checked="RadioButton_Checked"
                         Content="Normal"
                         GroupName="Group1"
                         IsChecked="True" />
            <RadioButton x:Name="Gray"
                         Checked="RadioButton_Checked"
                         Content="Gray"
                         GroupName="Group1" />
            <RadioButton x:Name="Canny"
                         Checked="RadioButton_Checked"
                         Content="Canny"
                         GroupName="Group1" />
            <RadioButton x:Name="Sepia"
                         Checked="RadioButton_Checked"
                         Content="Sepia"
                         GroupName="Group1" />
        </StackPanel>
    </Grid>

</phone:PhoneApplicationPage>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

