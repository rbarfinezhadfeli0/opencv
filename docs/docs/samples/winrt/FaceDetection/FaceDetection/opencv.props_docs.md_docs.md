# Documentation for `docs/samples/winrt/FaceDetection/FaceDetection/opencv.props_docs.md`

## File Metadata

- **Full Path**: `docs/samples/winrt/FaceDetection/FaceDetection/opencv.props_docs.md`
- **File Name**: `opencv.props_docs.md`
- **File Size**: 3,640 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/winrt/FaceDetection/FaceDetection/opencv.props_docs.md](../../../../../docs/samples/winrt/FaceDetection/FaceDetection/opencv.props_docs.md)

## Purpose and Role

This file is located in the `docs/samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/winrt/FaceDetection/FaceDetection/opencv.props`

## File Metadata

- **Full Path**: `samples/winrt/FaceDetection/FaceDetection/opencv.props`
- **File Name**: `opencv.props`
- **File Size**: 2,965 bytes
- **File Type**: .props
- **Link to Source**: [samples/winrt/FaceDetection/FaceDetection/opencv.props](../../../../samples/winrt/FaceDetection/FaceDetection/opencv.props)

## Purpose and Role

This file is located in the `samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ImportGroup Label="PropertySheets" />
  <PropertyGroup Label="UserMacros">
    <Runtime Condition="'$(ApplicationType)'=='Windows Phone'">WP</Runtime>
    <Runtime Condition="'$(ApplicationType)'=='Windows Store'">WS</Runtime>
    <OpenCV_Bin>$(OPENCV_WINRT_INSTALL_DIR)\$(Runtime)\8.1\$(PlatformTarget)\$(PlatformTarget)\vc12\bin\</OpenCV_Bin>
    <OpenCV_Lib>$(OPENCV_WINRT_INSTALL_DIR)\$(Runtime)\8.1\$(PlatformTarget)\$(PlatformTarget)\vc12\lib\</OpenCV_Lib>
    <OpenCV_Include>$(OPENCV_WINRT_INSTALL_DIR)\$(Runtime)\8.1\$(PlatformTarget)\include\</OpenCV_Include>
    <!--debug suffix for OpenCV dlls and libs -->
    <DebugSuffix Condition="'$(Configuration)'=='Debug'">d</DebugSuffix>
    <DebugSuffix Condition="'$(Configuration)'!='Debug'"></DebugSuffix>
  </PropertyGroup>
  <ItemGroup>
    <!--Add required OpenCV dlls here-->
    <None Include="$(OpenCV_Bin)opencv_core300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_imgcodecs300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_imgproc300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_features2d300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_flann300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_ml300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_objdetect300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_videoio300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
    <None Include="$(OpenCV_Bin)opencv_highgui300$(DebugSuffix).dll">
      <DeploymentContent>true</DeploymentContent>
    </None>
  </ItemGroup>
  <ItemDefinitionGroup>
    <ClCompile>
      <AdditionalIncludeDirectories>$(OpenCV_Include);%(AdditionalIncludeDirectories);</AdditionalIncludeDirectories>
    </ClCompile>
    <Link>
      <!--Add required OpenCV libs here-->
      <AdditionalDependencies>opencv_core300$(DebugSuffix).lib;opencv_imgproc300$(DebugSuffix).lib;opencv_imgcodecs300$(DebugSuffix).lib;opencv_flann300$(DebugSuffix).lib;opencv_videoio300$(DebugSuffix).lib;opencv_features2d300$(DebugSuffix).lib;opencv_objdetect300$(DebugSuffix).lib;opencv_ml300$(DebugSuffix).lib;opencv_highgui300$(DebugSuffix).lib;%(AdditionalDependencies)</AdditionalDependencies>
      <AdditionalLibraryDirectories>$(OpenCV_Lib);%(AdditionalLibraryDirectories);</AdditionalLibraryDirectories>
    </Link>
  </ItemDefinitionGroup>
</Project>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

