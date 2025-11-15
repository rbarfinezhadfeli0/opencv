# Documentation for `docs/samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest_docs.md`

## File Metadata

- **Full Path**: `docs/samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest_docs.md`
- **File Name**: `Package.appxmanifest_docs.md`
- **File Size**: 2,007 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest_docs.md](../../../../../docs/samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest_docs.md)

## Purpose and Role

This file is located in the `docs/samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest`

## File Metadata

- **Full Path**: `samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest`
- **File Name**: `Package.appxmanifest`
- **File Size**: 1,285 bytes
- **File Type**: .appxmanifest
- **Link to Source**: [samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest](../../../../samples/winrt/FaceDetection/FaceDetection/Package.appxmanifest)

## Purpose and Role

This file is located in the `samples/winrt/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿<?xml version="1.0" encoding="utf-8"?>
<Package xmlns="http://schemas.microsoft.com/appx/2010/manifest" xmlns:m2="http://schemas.microsoft.com/appx/2013/manifest">

  <Identity Name="f8308285-aea6-41b1-a76e-9954cfd46c7e"
            Publisher="CN=ericmitt"
            Version="1.0.0.0" />

  <Properties>
    <DisplayName>FaceDetection</DisplayName>
    <PublisherDisplayName>ericmitt</PublisherDisplayName>
    <Logo>Assets\StoreLogo.png</Logo>
  </Properties>

  <Prerequisites>
    <OSMinVersion>6.3.0</OSMinVersion>
    <OSMaxVersionTested>6.3.0</OSMaxVersionTested>
  </Prerequisites>

  <Resources>
    <Resource Language="x-generate"/>
  </Resources>

  <Applications>
    <Application Id="App"
        Executable="$targetnametoken$.exe"
        EntryPoint="FaceDetection.App">
        <m2:VisualElements
            DisplayName="FaceDetection"
            Square150x150Logo="Assets\Logo.png"
            Square30x30Logo="Assets\SmallLogo.png"
            Description="FaceDetection"
            ForegroundText="light"
            BackgroundColor="#464646">
            <m2:SplashScreen Image="Assets\SplashScreen.png" />
        </m2:VisualElements>
    </Application>
  </Applications>
  <Capabilities>
    <Capability Name="internetClient" />
  </Capabilities>
</Package>
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

