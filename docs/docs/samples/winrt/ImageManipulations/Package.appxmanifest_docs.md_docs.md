# Documentation for `docs/samples/winrt/ImageManipulations/Package.appxmanifest_docs.md`

## File Metadata

- **Full Path**: `docs/samples/winrt/ImageManipulations/Package.appxmanifest_docs.md`
- **File Name**: `Package.appxmanifest_docs.md`
- **File Size**: 2,559 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/winrt/ImageManipulations/Package.appxmanifest_docs.md](../../../../docs/samples/winrt/ImageManipulations/Package.appxmanifest_docs.md)

## Purpose and Role

This file is located in the `docs/samples/winrt/ImageManipulations` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/winrt/ImageManipulations/Package.appxmanifest`

## File Metadata

- **Full Path**: `samples/winrt/ImageManipulations/Package.appxmanifest`
- **File Name**: `Package.appxmanifest`
- **File Size**: 1,885 bytes
- **File Type**: .appxmanifest
- **Link to Source**: [samples/winrt/ImageManipulations/Package.appxmanifest](../../../samples/winrt/ImageManipulations/Package.appxmanifest)

## Purpose and Role

This file is located in the `samples/winrt/ImageManipulations` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Package xmlns="http://schemas.microsoft.com/appx/2010/manifest" xmlns:m2="http://schemas.microsoft.com/appx/2013/manifest">
  <Identity Name="Microsoft.SDKSamples.MediaCapture.CPP" Publisher="CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US" Version="1.1.0.6" />
  <Properties>
    <DisplayName>MediaCapture CPP sample</DisplayName>
    <PublisherDisplayName>Microsoft Corporation</PublisherDisplayName>
    <Logo>Assets\windows-sdk.png</Logo>
  </Properties>
  <Prerequisites>
    <OSMinVersion>6.3</OSMinVersion>
    <OSMaxVersionTested>6.3</OSMaxVersionTested>
  </Prerequisites>
  <Resources>
    <Resource Language="x-generate" />
  </Resources>
  <Applications>
    <Application Id="MediaCapture.App" Executable="$targetnametoken$.exe" EntryPoint="MediaCapture.App">
      <m2:VisualElements DisplayName="OCV Image Manipulations" Description="OpenCV Image Manipulations sample" BackgroundColor="#00b2f0" ForegroundText="light" Square150x150Logo="Assets\opencv-logo-150.png" Square30x30Logo="Assets\opencv-logo-30.png">
        <m2:DefaultTile ShortName="Ocv ImageManipulations">
          <m2:ShowNameOnTiles>
            <m2:ShowOn Tile="square150x150Logo" />
          </m2:ShowNameOnTiles>
        </m2:DefaultTile>
        <m2:SplashScreen BackgroundColor="#00b2f0" Image="Assets\splash-sdk.png" />
      </m2:VisualElements>
    </Application>
  </Applications>
  <Capabilities>
    <DeviceCapability Name="webcam" />
    <DeviceCapability Name="microphone" />
  </Capabilities>
  <Extensions>
    <Extension Category="windows.activatableClass.inProcessServer">
      <InProcessServer>
        <Path>OcvTransform.dll</Path>
        <ActivatableClass ActivatableClassId="OcvTransform.OcvImageManipulations" ThreadingModel="both" />
      </InProcessServer>
    </Extension>
  </Extensions>
</Package>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

