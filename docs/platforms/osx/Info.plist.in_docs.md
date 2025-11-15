# Documentation for `platforms/osx/Info.plist.in`

## File Metadata

- **Full Path**: `platforms/osx/Info.plist.in`
- **File Name**: `Info.plist.in`
- **File Size**: 623 bytes
- **File Type**: .in
- **Link to Source**: [platforms/osx/Info.plist.in](../../platforms/osx/Info.plist.in)

## Purpose and Role

This file is located in the `platforms/osx` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleName</key>
    <string>${OPENCV_APPLE_BUNDLE_NAME}</string>
    <key>CFBundleIdentifier</key>
    <string>${OPENCV_APPLE_BUNDLE_ID}</string>
    <key>CFBundleVersion</key>
    <string>${OPENCV_LIBVERSION}</string>
    <key>CFBundleShortVersionString</key>
    <string>${OPENCV_LIBVERSION}</string>
    <key>CFBundleSignature</key>
    <string>????</string>
    <key>CFBundlePackageType</key>
    <string>FMWK</string>
</dict>
</plist>

```

## General Information

This file is part of the OpenCV repository infrastructure.

