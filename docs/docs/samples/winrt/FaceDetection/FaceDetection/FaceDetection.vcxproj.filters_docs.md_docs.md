# Documentation for `docs/samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters_docs.md`

## File Metadata

- **Full Path**: `docs/samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters_docs.md`
- **File Name**: `FaceDetection.vcxproj.filters_docs.md`
- **File Size**: 3,146 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters_docs.md](../../../../../docs/samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters_docs.md)

## Purpose and Role

This file is located in the `docs/samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters`

## File Metadata

- **Full Path**: `samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters`
- **File Name**: `FaceDetection.vcxproj.filters`
- **File Size**: 2,384 bytes
- **File Type**: .filters
- **Link to Source**: [samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters](../../../../samples/winrt/FaceDetection/FaceDetection/FaceDetection.vcxproj.filters)

## Purpose and Role

This file is located in the `samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="12.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <Filter Include="Common">
      <UniqueIdentifier>0fae44e0-6c15-4cca-abda-29556b391390</UniqueIdentifier>
    </Filter>
    <Filter Include="Assets">
      <UniqueIdentifier>cee6a8fd-c14c-4eab-b410-54073b7fc217</UniqueIdentifier>
      <Extensions>bmp;fbx;gif;jpg;jpeg;tga;tiff;tif;png</Extensions>
    </Filter>
    <Image Include="Assets\Logo.scale-100.png">
      <Filter>Assets</Filter>
    </Image>
    <Image Include="Assets\SmallLogo.scale-100.png">
      <Filter>Assets</Filter>
    </Image>
    <Image Include="Assets\StoreLogo.scale-100.png">
      <Filter>Assets</Filter>
    </Image>
    <Image Include="Assets\SplashScreen.scale-100.png">
      <Filter>Assets</Filter>
    </Image>
  </ItemGroup>
  <ItemGroup>
    <ApplicationDefinition Include="App.xaml" />
  </ItemGroup>
  <ItemGroup>
    <ClCompile Include="App.xaml.cpp" />
    <ClCompile Include="MainPage.xaml.cpp" />
    <ClCompile Include="pch.cpp" />
  </ItemGroup>
  <ItemGroup>
    <ClInclude Include="pch.h" />
    <ClInclude Include="App.xaml.h" />
    <ClInclude Include="MainPage.xaml.h" />
  </ItemGroup>
  <ItemGroup>
    <AppxManifest Include="Package.appxmanifest" />
  </ItemGroup>
  <ItemGroup>
    <None Include="FaceDetection_TemporaryKey.pfx" />
    <None Include="$(OpenCV_Bin)opencv_core300$(DebugSuffix).dll" />
    <None Include="$(OpenCV_Bin)opencv_imgproc300$(DebugSuffix).dll" />
    <None Include="$(OpenCV_Bin)opencv_features2d300$(DebugSuffix).dll" />
    <None Include="$(OpenCV_Bin)opencv_flann300$(DebugSuffix).dll" />
    <None Include="$(OpenCV_Bin)opencv_ml300$(DebugSuffix).dll" />
    <None Include="$(OpenCV_Bin)opencv_objdetect300$(DebugSuffix).dll" />
    <None Include="$(OpenCV_Bin)opencv_imgcodecs300$(DebugSuffix).dll" />
  </ItemGroup>
  <ItemGroup>
    <Page Include="MainPage.xaml" />
  </ItemGroup>
  <ItemGroup>
    <Image Include="Assets\group1.jpg">
      <Filter>Assets</Filter>
    </Image>
    <Image Include="Assets\group2.JPG">
      <Filter>Assets</Filter>
    </Image>
    <Image Include="Assets\group3.jpg">
      <Filter>Assets</Filter>
    </Image>
  </ItemGroup>
  <ItemGroup>
    <Xml Include="Assets\haarcascade_frontalface_alt.xml">
      <Filter>Assets</Filter>
    </Xml>
  </ItemGroup>
</Project>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

