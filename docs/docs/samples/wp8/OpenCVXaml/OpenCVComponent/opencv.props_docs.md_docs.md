# Documentation for `docs/samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props_docs.md`

## File Metadata

- **Full Path**: `docs/samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props_docs.md`
- **File Name**: `opencv.props_docs.md`
- **File Size**: 2,423 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props_docs.md](../../../../../docs/samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props_docs.md)

## Purpose and Role

This file is located in the `docs/samples/wp8/OpenCVXaml/OpenCVComponent` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props`

## File Metadata

- **Full Path**: `samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props`
- **File Name**: `opencv.props`
- **File Size**: 1,763 bytes
- **File Type**: .props
- **Link to Source**: [samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props](../../../../samples/wp8/OpenCVXaml/OpenCVComponent/opencv.props)

## Purpose and Role

This file is located in the `samples/wp8/OpenCVXaml/OpenCVComponent` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ImportGroup Label="PropertySheets" />
  <PropertyGroup Label="UserMacros">
    <OpenCV_Bin>$(OPENCV_WINRT_INSTALL_DIR)\WP\8.0\$(PlatformTarget)\$(PlatformTarget)\vc11\bin\</OpenCV_Bin>
    <OpenCV_Lib>$(OPENCV_WINRT_INSTALL_DIR)\WP\8.0\$(PlatformTarget)\$(PlatformTarget)\vc11\lib\</OpenCV_Lib>
    <OpenCV_Include>$(OPENCV_WINRT_INSTALL_DIR)\WP\8.0\$(PlatformTarget)\include\</OpenCV_Include>
    <!--debug suffix for OpenCV dlls and libs -->
    <DebugSuffix Condition="'$(Configuration)'=='Debug'">d</DebugSuffix>
    <DebugSuffix Condition="'$(Configuration)'!='Debug'"></DebugSuffix>
  </PropertyGroup>
  <PropertyGroup>
    <IgnoreImportLibrary>true</IgnoreImportLibrary>
  </PropertyGroup>
  <ItemGroup>
    <!--Add required OpenCV dlls here-->
    <None Include="$(OpenCV_Bin)opencv_core300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_imgproc300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
  </ItemGroup>
  <ItemDefinitionGroup>
    <ClCompile>
      <AdditionalIncludeDirectories>$(OpenCV_Include);%(AdditionalIncludeDirectories);</AdditionalIncludeDirectories>
    </ClCompile>
    <Link>
      <!--Add required OpenCV libs here-->
      <AdditionalDependencies>opencv_core300$(DebugSuffix).lib;opencv_imgproc300$(DebugSuffix).lib;WindowsPhoneCore.lib;RuntimeObject.lib;PhoneAppModelHost.lib;%(AdditionalDependencies)</AdditionalDependencies>
      <AdditionalLibraryDirectories>$(OpenCV_Lib);%(AdditionalLibraryDirectories);</AdditionalLibraryDirectories>
    </Link>
  </ItemDefinitionGroup>
</Project>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

