# Documentation for `samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.Shared/video_capture_xaml.Shared.vcxitems`

## File Metadata

- **Full Path**: `samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.Shared/video_capture_xaml.Shared.vcxitems`
- **File Name**: `video_capture_xaml.Shared.vcxitems`
- **File Size**: 1,610 bytes
- **File Type**: .vcxitems
- **Link to Source**: [samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.Shared/video_capture_xaml.Shared.vcxitems](../../../../../samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.Shared/video_capture_xaml.Shared.vcxitems)

## Purpose and Role

This file is located in the `samples/winrt_universal/VideoCaptureXAML/video_capture_xaml/video_capture_xaml.Shared` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup Label="Globals">
    <MSBuildAllProjects>$(MSBuildAllProjects);$(MSBuildThisFileFullPath)</MSBuildAllProjects>
    <HasSharedItems>true</HasSharedItems>
    <SharedGUID>98633655-f156-43bb-b452-37cd6a71e3f0</SharedGUID>
    <ItemsProjectGuid>{6a274b7f-3982-499e-b55a-1f12ef2e3ec0}</ItemsProjectGuid>
    <ItemsRootNamespace>highgui_xaml</ItemsRootNamespace>
    <ItemsProjectName>video_capture_xaml.Shared</ItemsProjectName>
  </PropertyGroup>
  <ItemDefinitionGroup>
    <ClCompile>
      <AdditionalIncludeDirectories>%(AdditionalIncludeDirectories);$(MSBuildThisFileDirectory)</AdditionalIncludeDirectories>
    </ClCompile>
  </ItemDefinitionGroup>
  <ItemGroup>
    <ApplicationDefinition Include="$(MSBuildThisFileDirectory)App.xaml">
      <SubType>Designer</SubType>
    </ApplicationDefinition>
    <ClCompile Include="$(MSBuildThisFileDirectory)App.xaml.cpp">
      <DependentUpon>$(MSBuildThisFileDirectory)App.xaml</DependentUpon>
    </ClCompile>
    <ClCompile Include="$(MSBuildThisFileDirectory)main.cpp" />
    <ClInclude Include="$(MSBuildThisFileDirectory)App.xaml.h">
      <DependentUpon>$(MSBuildThisFileDirectory)App.xaml</DependentUpon>
    </ClInclude>
    <ClCompile Include="$(MSBuildThisFileDirectory)pch.cpp">
      <PrecompiledHeader>Create</PrecompiledHeader>
    </ClCompile>
    <ClInclude Include="$(MSBuildThisFileDirectory)pch.h" />
  </ItemGroup>
  <ItemGroup>
    <ProjectCapability Include="SourceItemsFromImports" />
  </ItemGroup>
</Project>
```

## General Information

This file is part of the OpenCV repository infrastructure.

