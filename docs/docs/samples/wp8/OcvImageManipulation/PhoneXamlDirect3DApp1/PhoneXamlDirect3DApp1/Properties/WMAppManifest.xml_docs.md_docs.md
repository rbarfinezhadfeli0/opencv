# Documentation for `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml_docs.md`
- **File Name**: `WMAppManifest.xml_docs.md`
- **File Size**: 3,422 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml_docs.md](../../../../../../../docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml`

## File Metadata

- **Full Path**: `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml`
- **File Name**: `WMAppManifest.xml`
- **File Size**: 2,123 bytes
- **File Type**: .xml
- **Link to Source**: [samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml](../../../../../../samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties/WMAppManifest.xml)

## Purpose and Role

This file is located in the `samples/wp8/OcvImageManipulation/PhoneXamlDirect3DApp1/PhoneXamlDirect3DApp1/Properties` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Deployment xmlns="http://schemas.microsoft.com/windowsphone/2012/deployment" AppPlatformVersion="8.0">
  <DefaultLanguage xmlns="" code="en-US" />
  <App xmlns="" ProductID="{cc734b3d-d8f2-4528-b223-0e7b8a4f6cc7}" Title="OCVImageManipulation" RuntimeType="Silverlight" Version="1.0.0.0" Genre="apps.normal" Author="Microsoft Open Technologies, Inc." Description="Sample description" Publisher="Microsoft Open Technologies, Inc." PublisherID="{4029c95e-d442-45ca-b556-33d7e3bde613}">
    <IconPath IsRelative="true" IsResource="false">Assets\ApplicationIcon.png</IconPath>
    <Capabilities>
      <Capability Name="ID_CAP_SENSORS" />
      <Capability Name="ID_CAP_WEBBROWSERCOMPONENT" />
      <Capability Name="ID_CAP_MEDIALIB_PLAYBACK" />
      <Capability Name="ID_CAP_MEDIALIB_PHOTO" />
      <Capability Name="ID_CAP_MEDIALIB_AUDIO" />
      <Capability Name="ID_CAP_NETWORKING" />
      <Capability Name="ID_CAP_ISV_CAMERA" />
    </Capabilities>
    <Tasks>
      <DefaultTask Name="_default" NavigationPage="MainPage.xaml" />
    </Tasks>
    <Tokens>
      <PrimaryToken TokenID="PhoneXamlDirect3DApp1Token" TaskName="_default">
        <TemplateFlip>
          <SmallImageURI IsRelative="true" IsResource="false">Assets\Tiles\FlipCycleTileSmall.png</SmallImageURI>
          <Count>0</Count>
          <BackgroundImageURI IsRelative="true" IsResource="false">Assets\Tiles\FlipCycleTileMedium.png</BackgroundImageURI>
          <Title>OCVImageManipulation</Title>
          <BackContent>
          </BackContent>
          <BackBackgroundImageURI>
          </BackBackgroundImageURI>
          <BackTitle>
          </BackTitle>
          <DeviceLockImageURI>
          </DeviceLockImageURI>
          <HasLarge>
          </HasLarge>
        </TemplateFlip>
      </PrimaryToken>
    </Tokens>
    <ScreenResolutions>
      <ScreenResolution Name="ID_RESOLUTION_WVGA" />
      <ScreenResolution Name="ID_RESOLUTION_WXGA" />
      <ScreenResolution Name="ID_RESOLUTION_HD720P" />
    </ScreenResolutions>
  </App>
</Deployment>
```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

